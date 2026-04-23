# GitHub Actions CI/CD — Mental Model

## Why This Document Exists

When I took over the `agentic_analytics_migrator` repo, one of the first things I noticed was a `.github/workflows/deploy.yml` sitting in the root. I knew what it was for — CI/CD — but I didn't have a clear picture of how it actually worked. I could read the lines, but I couldn't picture the components talking to each other: where do secrets come from, who reads them, where do they end up, and how does pushing code eventually result in a running container in Azure?

So I worked through it by asking questions as I went — debugging a broken pipeline, chasing missing secrets, tracing values from GitHub all the way into a running container app. This document captures what I built in my head through that process.

The goal isn't to document the specific project — it's to have a reusable mental model for any repo I take over that has a GitHub Actions CI/CD pipeline. If I ever see a `deploy.yml` again and feel the same blankness, I read this first.

---

## The Big Picture (One Flow to Rule Them All)

```
GitHub Secrets (store credentials and sensitive values)
        ↓
Pipeline reads ${{ secrets.* }} → authenticates to Azure
        ↓
Builds Docker image → pushes to ACR (Azure Container Registry)
        ↓
Deploys image to Azure Container Apps
        ↓
Copies relevant secrets as container env vars
        ↓
App runs in Azure — reads those env vars at runtime
```

This pattern is the same regardless of platform (GitHub Actions, Azure DevOps, GitLab CI, Jenkins). The technology changes, the mental model stays.

---

## Key Insight #1 — Secrets Live in GitHub, Not Azure

When you see `${{ secrets.AZURE_CREDENTIALS }}` in a workflow file, that value is stored in:

**GitHub → Repo → Settings → Secrets and variables → Actions → Repository secrets**

It is *not* in Azure. It's a GitHub-native feature. The value inside the secret *happens to be* Azure credentials, but GitHub is the vault.

### The three scopes of GitHub Secrets (in order of visibility):
| Scope | Who can see it |
|---|---|
| Repository secrets | Only that repo |
| Organization secrets | All repos in the org — but only visible to org admins |
| Environment secrets | Only when a GitHub Environment is configured in the workflow |

**Why this matters:** If your repo shows "no repository secrets" but the pipeline worked before, the secrets are likely at the **organization level** — invisible to you unless you're an org admin.

---

## Key Insight #2 — Secrets Get Copied Into Container Env Vars

When the workflow runs `az containerapp update --set-env-vars`, it's writing secret values directly into the running container as environment variables. Example:

```yaml
--set-env-vars \
  AZURE_OPENAI_API_KEY="${{ secrets.AZURE_AI_API_KEY }}" \
  POSTGRES_DSN="${{ secrets.POSTGRES_DSN }}"
```

This means:

- After a successful deployment, the container has those values baked in as env vars
- You can **retrieve secret values from the running container** if you have Azure portal access — even if you've lost the original GitHub Secret values
- Azure Portal → Container App → Environment Variables tab shows what's currently set

### The one exception: `AZURE_CREDENTIALS`
This secret is only used by the pipeline to authenticate to Azure during deploy. It is **never copied** to any container — the running app has no need for it.

---

## Key Insight #3 — Two Types of Values in the Workflow

Not everything in a workflow is a secret. Compare:

```yaml
env:                            # Plain text — safe to hardcode
  RESOURCE_GROUP: agentic-migration-app
  ACR_NAME: agenticmigapp

secrets:                        # Sensitive — must be in GitHub Secrets
  AZURE_CREDENTIALS: ...
  AZURE_AI_API_KEY: ...
  POSTGRES_DSN: ...
```

**Rule of thumb:** If knowing the value would give someone access to something (pay for API calls, connect to a DB, impersonate the app) → secret. If it's just a name or identifier → plain text env var in the workflow.

---

## Key Insight #4 — CI vs CD Are Different Phases

| Phase | What happens | Example in deploy.yml |
|---|---|---|
| **CI** (Continuous Integration) | Validate, build, and package the code | `docker build`, `docker push` to ACR |
| **CD** (Continuous Deployment) | Deliver the built artifact to an environment | `az containerapp update`, setting env vars, verify-deployment job |

Your `deploy.yml` does both in one file. Some projects split these into separate files (e.g., `build-deploy-backend.yml`, `build-deploy-frontend.yml`) for more granular control.

---

## Key Insight #5 — No Azure Key Vault ≠ Insecure, Just Different

The current setup uses `--set-env-vars` which stores values as **plain text** in the Container App configuration. A more secure approach uses Azure Container Apps' own secrets store + `secretref:`:

```
Current:   GitHub Secret → pipeline → env var (plain text in Azure)
Secure:    GitHub Secret → pipeline → Container App Secret store → env var (masked in Azure)
Even more: Azure Key Vault → pipeline fetches at runtime → never stored in GitHub at all
```

How to check whether a project uses Key Vault — look in the workflow for:
- `azure/get-keyvault-secrets` action
- Any reference to a vault URL or `keyvault` keyword

If you only see `${{ secrets.* }}` — it's GitHub-native secrets only.

---

## Key Insight #6 — Secrets Don't Migrate With Code

When a repo is migrated or forked, **secrets do not transfer**. Code comes over, secrets stay behind. This is a common reason a pipeline that worked in the old repo fails in the new one:

1. Secrets existed in old repo → pipeline worked
2. Code migrated to new repo → secrets not migrated
3. New repo shows "no repository secrets" → pipeline fails at `azure/login@v2`

**Fix:** Ask the person who set up the original repo to re-add the secrets. For `AZURE_CREDENTIALS` specifically — if the original JSON value is lost, a new client secret must be generated from Azure AD (App Registrations → the app → Certificates & secrets → New client secret). The `clientId` and `tenantId` are still visible in the app overview; only the `clientSecret` value needs to be regenerated.

---

## Key Insight #7 — Understanding azure/login@v2

The `AZURE_CREDENTIALS` JSON that GitHub Secrets stores doesn't come out of thin air — it originates from an **Azure App Registration**.

### Where it comes from

Someone (usually a DevOps engineer or Azure admin) goes into Azure AD, creates an App Registration, and registers it as a service principal — essentially a dedicated identity for the pipeline to use. Once that app registration exists, they generate a **client secret** from it, then package everything into a JSON object and paste it into GitHub Secrets as `AZURE_CREDENTIALS`.

```
Azure AD → App Registrations → Create app → Generate client secret
                                                    ↓
                              Combine into JSON → paste into GitHub Secrets
                                                    ↓
                              deploy.yml reads it → azure/login@v2 authenticates to Azure
```

### The JSON format

```json
{
  "clientId": "...",
  "clientSecret": "...",
  "tenantId": "...",
  "subscriptionId": "..."
}
```

This is a **service principal** — an identity in Azure AD created specifically for automated pipelines, scoped only to the resources it needs (e.g., a specific resource group). It's the Azure equivalent of a GitHub bot account.

### Where to find each piece

| Value | Where to find it |
|---|---|
| `clientId` | Azure AD → App Registrations → the app → Overview → Application (client) ID |
| `tenantId` | Azure AD → App Registrations → the app → Overview → Directory (tenant) ID |
| `subscriptionId` | Azure Portal → Subscriptions |
| `clientSecret` | One-time reveal only — must generate a new one if lost |

### Important: clientSecret is a one-time reveal

When you generate a client secret from the App Registration, Azure shows you the value **once**. After you navigate away, it's masked forever. This is why losing `AZURE_CREDENTIALS` requires generating a *new* client secret from the same App Registration — you can't recover the old one. The `clientId` and `tenantId` are always visible in the app overview, so only the `clientSecret` needs to be regenerated.

---

## The Full Secret Inventory (for this project)

| Secret Name | Purpose | Goes to container? |
|---|---|---|
| `AZURE_CREDENTIALS` | Pipeline authenticates to Azure | No — pipeline only |
| `AZURE_AI_ENDPOINT` | Azure OpenAI endpoint URL | Yes — backend |
| `AZURE_AI_API_KEY` | Azure OpenAI API key | Yes — backend |
| `AZURE_AI_DEPLOYMENT_NAME` | Model deployment name | Yes — backend |
| `AZURE_AI_API_VERSION` | API version | Yes — backend |
| `POSTGRES_DSN` | PostgreSQL connection string | Yes — backend |
| `APP_DB_URL` | App state DB URL | Yes — backend |
| `BASIC_AUTH_USERNAME` | App login username | Yes — both |
| `BASIC_AUTH_PASSWORD` | App login password | Yes — both |

---

## Platform Equivalents (Same Concept, Different Names)

| Concept | GitHub Actions | Azure DevOps | GitLab CI | Jenkins |
|---|---|---|---|---|
| Secret store | GitHub Secrets | Variable Groups | CI/CD Variables | Credentials |
| Secret reference | `${{ secrets.X }}` | `$(X)` | `$X` | `credentials('X')` |
| Pipeline file | `deploy.yml` | `azure-pipelines.yml` | `.gitlab-ci.yml` | `Jenkinsfile` |
| Artifact registry | ACR / Docker Hub | ACR / Docker Hub | GitLab Registry | Nexus |
| Cloud login step | `azure/login@v2` | Azure service connection | Environment | Plugin |

Azure DevOps has one advantage: native Key Vault integration — you can link a Variable Group directly to a Key Vault so secrets are pulled from there at pipeline runtime, with no manual copying.

---

## Bonus — Terraform vs Bicep vs Manual (Infrastructure as Code)

This project set up Azure resources manually (via portal or CLI). Other teams use IaC tools instead:

| | Manual | Bicep | Terraform |
|---|---|---|---|
| Resources defined | In your head / portal clicks | `.bicep` files | `.tf` files |
| Azure-native | Yes | Yes | Via provider plugin |
| Multi-cloud | No | No | Yes |
| Recreate from scratch | Painful | Run the file | `terraform apply` |
| Preview changes | Not possible | `what-if` flag | `terraform plan` |

In a project with `terraform-plan.yml` and `terraform-apply.yml`, those workflows manage the Azure infrastructure itself (Container Apps, ACR, networking). The `build-deploy-*.yml` files then deploy the app on top of that infrastructure.

---

## Quick Diagnostic Checklist (for future repo takeovers)

When you open a new repo with a CI/CD pipeline:

- [ ] What workflow files exist in `.github/workflows/`?
- [ ] What secrets does it reference (`grep -i "secrets\."`)? 
- [ ] Are those secrets at repo level, org level, or environment level?
- [ ] Does it use Key Vault, or GitHub-native secrets only?
- [ ] Which secrets get copied to container env vars vs. stay in the pipeline?
- [ ] Are the Azure resources provisioned via IaC (Terraform/Bicep) or manually?
- [ ] Who owns the service principal / Azure AD app registration?
- [ ] When does the service principal secret expire?

---

## Self-Assessment — Is This Mental Model Good?

### What's strong

The instinct to trace the full chain — from where a value is stored, to who reads it, to where it ends up — is exactly the right way to understand a system you didn't build. Most people stop at "it's in secrets" without asking what that actually means or where those values go after deployment.

The third observation (realizing you can recover secret values from the container env vars in Azure instead of asking the old owner) shows active learning, not passive reading. That's looking for leverage from what you're discovering — an architect-level habit, not just an engineer habit.

### One gap worth closing

The mental model currently ends at "secrets copied to container env vars." The next layer is what happens inside the container at runtime — how the app actually reads those env vars:

```python
import os
api_key = os.environ.get("AZURE_OPENAI_API_KEY")
```

And what happens if one is missing or wrong — does the app crash on startup, fail silently, or throw at the point of use? That's less CI/CD and more application runtime behavior, but it completes the full picture from pipeline to running code.

### The bigger pattern

What's being built here — not just learning a tool but extracting a transferable mental model — is the right habit. The diagnostic checklist above is proof of that. This document isn't "how this one deploy.yml works." It's "how to approach any deploy.yml." That's the difference between knowledge and a thinking model.

The way this understanding was built — debugging a live broken pipeline, tracing values, asking why at each step — is also the right method. Keep doing it this way.
