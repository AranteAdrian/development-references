# UAT / Prod Access Denied -- Ingress IP Restrictions

## What You See

- You open the backend URL for UAT or Prod in your browser
- Instead of a login popup, you get an "RBAC access denied" or similar error
- You never get a chance to enter credentials
- Dev works fine with no issues

## Why It Happens

UAT and Prod have **IP security restrictions** enabled on the Azure Container Apps ingress. This is a security measure that only allows traffic from approved IP addresses. If your IP is not on the allow list, Azure blocks the request before it even reaches the application.

Dev does not have these restrictions, which is why it works from anywhere.

This is expected and correct -- UAT and Prod should be more locked down than Dev.

## How to Fix It

### Step 1: Get your public IP address

PowerShell:
```powershell
(Invoke-WebRequest -Uri "https://api.ipify.org").Content
```

Command Prompt:
```cmd
curl https://api.ipify.org
```

### Step 2: Add your IP to the allow list

You need Contributor access to the Azure subscription for the environment you want to access.

**Option A -- Azure Portal:**
1. Go to https://portal.azure.com
2. Search for the Container App (e.g. `ac-ai-fans-uat-ingenious-backend`)
3. Go to **Settings** > **Ingress**
4. Under **IP Security Restrictions**, add your IP with `/32` suffix (e.g. `203.32.108.4/32`)
5. Set the action to **Allow**
6. Save

**Option B -- Azure CLI:**
```bash
az containerapp ingress access-restriction set \
  --name ac-ai-fans-uat-ingenious-backend \
  --resource-group ai-promt-tuner \
  --rule-name "YourName" \
  --action Allow \
  --ip-address YOUR_IP/32
```

### Step 3: Test

Open the backend URL in your browser. You should now see the Basic Auth login popup instead of the access denied error.

## Important Notes

- Your home IP address may change over time (dynamic IP). If you suddenly lose access, check if your IP has changed and update the allow list.
- If you're on a corporate VPN or office network, ask the network team for the static IP range and add that instead of your individual IP.
- If you don't have Contributor access to add your own IP, ask a team member who does.
