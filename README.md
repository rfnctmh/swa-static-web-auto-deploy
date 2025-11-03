# Azure Static Web Apps Demo (Auto Deploy + Last Build Time)

This project demonstrates:
- Auto deploy to Azure Static Web Apps on every push
- Daily **on-time** redeploy (choose one):
  - GitHub Actions schedule (may delay)
  - Azure Functions Timer → trigger `workflow_dispatch` (precise)
- Frontend shows the latest build time from `build.txt` (Taiwan time, GMT+8)

## Usage
1. Push this repo to your GitHub account.
2. Create an Azure Static Web App and link to this repo (app location = `site`).
3. Secrets:
   - If using **deployment token**: add `AZURE_STATIC_WEB_APPS_API_TOKEN` in GitHub → Settings → Secrets → Actions.
   - If using **OIDC (default workflow)**: token secret is auto-managed by Azure; no change needed.
4. On each push (and on your daily trigger), GitHub Actions will:
   - write `site/build.txt` with **Taiwan time**:
     ```
     Last build: YYYY-MM-DD HH:MM:SS GMT+8
     ```
   - deploy to SWA.
5. The page fetches `build.txt` and **displays its full content** (no parsing needed).

## Trigger options
- **GitHub Actions schedule (simple)**
  - `on.schedule: "0 0 * * *"` (runs near 08:00 TW; may delay)
- **Azure Functions Timer (recommended for precise 08:00 TW)**
  - Function calls GitHub REST API `workflow_dispatch` at 00:00 UTC (08:00 TW).

## Notes
- If you switch to Azure Functions for scheduling, **remove** or comment out the `schedule:` block in the workflow to avoid double deployments.
- In DNS/Custom domains, use CNAME validation per subdomain (`www`, `app`, `blog`, …); Cloudflare users should set records to **DNS only (grey cloud)** during validation.
