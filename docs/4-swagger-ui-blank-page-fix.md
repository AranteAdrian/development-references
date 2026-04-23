# Swagger UI Blank Page Fix (Deployed Environments)

## The Problem

When opening the backend Swagger UI (`/docs`) in deployed environments, different issues appeared depending on the environment:

| Environment | Behaviour | Root Cause |
|-------------|-----------|------------|
| Local | Works fine | No restrictions on loading external resources |
| Dev | Blank page after login | Content Security Policy blocks external CDN |
| UAT / Prod | "RBAC access denied" | Azure Easy Auth enabled at infrastructure level |

This document covers the **Dev blank page fix**. The UAT/Prod RBAC issue is a separate Azure infrastructure problem (see bottom of this doc).

---

## Understanding the Root Cause (Dev Blank Page)

### What is Swagger UI?

Swagger UI is the interactive API documentation page you see at `/docs`. It is built into FastAPI and loads automatically when you start the backend.

### How Swagger UI Loads (Normally)

```
1. Browser requests /docs from the backend
2. Backend returns a small HTML page
3. That HTML page tells the browser to load two files from an external CDN:
   - swagger-ui-bundle.js  (the JavaScript that powers Swagger)
   - swagger-ui.css        (the styling)
4. Browser downloads those files from cdn.jsdelivr.net
5. Swagger UI renders in the browser
```

### What is a Content Security Policy (CSP)?

A CSP is a security header that the server sends with every response. It acts like a whitelist -- it tells the browser which sources are allowed to load resources (scripts, styles, images, etc.).

Think of it as a **bouncer at a door**: "Only allow resources from THESE approved sources. Block everything else."

### What Went Wrong

The deployed backend sends a CSP header that says:

```
script-src 'self'   -->  Only load JavaScript from our own domain
style-src 'self'    -->  Only load CSS from our own domain
```

But Swagger UI tries to load JS and CSS from `cdn.jsdelivr.net`, which is NOT our domain. The browser obeys the CSP and blocks those files. Result: the HTML shell loads, but without JS or CSS, the page is blank.

### Why It Works Locally

The local backend does not send this CSP header. Without a CSP, the browser has no restrictions and loads the CDN files without any issues.

---

## The Solution

We followed [FastAPI's official recommendation](https://fastapi.tiangolo.com/how-to/custom-docs-ui-assets/) to **self-host the Swagger UI assets** instead of loading them from the CDN.

### What We Changed

#### 1. Downloaded the Swagger UI Files

We saved the two files that Swagger UI needs into the backend project:

```
backend/
  static/
    swagger-ui-bundle.js   (~1.5 MB)
    swagger-ui.css          (~179 KB)
```

These are the exact same files that would normally load from `cdn.jsdelivr.net`, but now they live inside our project.

#### 2. Modified the Backend Code

**File:** `backend/ingenious_extensions/api/routes/custom.py`

Changes made:

- **Removed the default `/docs` route** that FastAPI creates (which points to the CDN)
- **Mounted the `static/` directory** so the backend can serve the Swagger UI files itself
- **Added a custom `/docs` route** that points to `/static/swagger-ui-bundle.js` and `/static/swagger-ui.css` instead of the CDN
- **Exempted `/static` from Basic Auth** so the browser can load the JS/CSS files after the user authenticates for `/docs`

### Before vs After

```
BEFORE (broken in deployed):
  /docs HTML --> loads JS/CSS from cdn.jsdelivr.net --> BLOCKED by CSP --> blank page

AFTER (fixed):
  /docs HTML --> loads JS/CSS from /static/ (same domain) --> ALLOWED by CSP --> Swagger renders
```

### Authentication Flow (Unchanged)

The fix does not change how authentication works:

```
1. User opens /docs in the browser
2. Backend returns 401 (Unauthorized) with a login challenge
3. Browser shows the native username/password popup
4. User enters credentials
5. Browser caches credentials and re-sends the request
6. /docs HTML loads successfully
7. Browser fetches /static/swagger-ui-bundle.js (exempt from auth, allowed by CSP)
8. Browser fetches /static/swagger-ui.css (exempt from auth, allowed by CSP)
9. Browser fetches /openapi.json (auth credentials sent automatically by browser)
10. Swagger UI renders fully
```

---

## Key Files

| File | Purpose |
|------|---------|
| `backend/ingenious_extensions/api/routes/custom.py` | Custom docs setup and auth middleware |
| `backend/static/swagger-ui-bundle.js` | Self-hosted Swagger UI JavaScript |
| `backend/static/swagger-ui.css` | Self-hosted Swagger UI CSS |

---

## How to Verify

After deploying, run these checks:

```bash
# 1. Swagger UI should return 200 (after auth)
curl -u "username:password" -s -o /dev/null -w "%{http_code}" https://<backend-url>/docs
# Expected: 200

# 2. Static assets should return 200 (no auth needed)
curl -s -o /dev/null -w "%{http_code}" https://<backend-url>/static/swagger-ui-bundle.js
# Expected: 200

# 3. OpenAPI schema should return 200 (after auth)
curl -u "username:password" -s -o /dev/null -w "%{http_code}" https://<backend-url>/openapi.json
# Expected: 200

# 4. Protected endpoints should still require auth
curl -s -o /dev/null -w "%{http_code}" https://<backend-url>/api/v1/revisions/list
# Expected: 401
```

Or simply open `/docs` in the browser, enter credentials when prompted, and confirm Swagger renders with no blank page.

---

## How to Update the Swagger UI Version

If you need to update the Swagger UI version in the future:

1. Download the latest files:
   ```bash
   cd backend/static
   curl -L "https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui-bundle.js" -o swagger-ui-bundle.js
   curl -L "https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui.css" -o swagger-ui.css
   ```

2. Commit and deploy. No code changes needed.

---

## UAT/Prod "RBAC Access Denied" (Separate Issue)

The "RBAC access denied" error on UAT and Prod is NOT related to the Swagger UI fix above. It is caused by **Azure Container Apps built-in authentication (Easy Auth)** being enabled at the Azure infrastructure level.

When Easy Auth is enabled, Azure blocks all unauthenticated requests before they reach the application. This means the browser never gets to the backend's own login challenge.

### To Diagnose

```bash
# Check if Easy Auth is enabled
az containerapp auth show \
  --name <container-app-name> \
  --resource-group ai-promt-tuner
```

### To Fix

Either set to AllowAnonymous (let the app handle auth):
```bash
az containerapp auth update \
  --name <container-app-name> \
  --resource-group ai-promt-tuner \
  --unauthenticated-client-action AllowAnonymous
```

Or disable Easy Auth entirely (if app-level Basic Auth is sufficient):
```bash
az containerapp auth update \
  --name <container-app-name> \
  --resource-group ai-promt-tuner \
  --enabled false
```

This must be done for both UAT and Prod container apps before promoting the code fix.
