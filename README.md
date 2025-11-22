# Azure Static Web Apps Demo (Auto Deploy + Last Build Time)

This project demonstrates:

- Auto deploy to Azure Static Web Apps on every push
- Daily **on-time** redeploy by:
  - GitHub Actions schedule (may have some delay)
  - (Planned) Azure Functions Timer + Google Cloud Scheduler
- Frontend reads the latest build info from `site/build.txt` (Taiwan time, GMT+8),  
  including **when** and **who triggered** the deployment.

---

## Usage

1. Push this repo to your GitHub account.
2. Create an Azure Static Web App and link it to this repo.
   - App location: `site`
   - API location: *(empty)*
3. Secrets:
   - If using **deployment token**:
     - In GitHub → Settings → Secrets → Actions, add  
       `AZURE_STATIC_WEB_APPS_API_TOKEN`.
   - If using **OIDC (default template)**:
     - Azure + GitHub handle the token automatically, no extra secret needed.
4. On each push / PR / schedule run, GitHub Actions will:
   - Check out this repo
   - Write the latest build info to `site/build.txt`, including the event source.
   - Deploy `site/` to Azure Static Web Apps.
5. The page fetches `build.txt` and writes its **full text** into the `#build-time` element.
   - Currently this element is hidden:
     ```html
     <p id="build-time" style="display:none;"></p>
     ```
   - If you want to show it on the page, simply remove the `display:none` and style it as you like.

---

## Trigger options (current)

Right now the workflow supports:

- **Push / Pull Request**
  - `on.push` to `main` / `master`
  - `on.pull_request` (opened, synchronize, reopened) to `main` / `master`
- **GitHub Actions schedule**
  - `on.schedule: "0 0 * * *"`  
    → runs once a day, around 08:00 (Taipei, UTC+8)
- **Manual trigger**
  - `on.workflow_dispatch`  
    → run from GitHub UI or API

In all cases, the `Write build time file` step writes:

```text
Last build: YYYY-MM-DD HH:MM:SS GMT+8 Source: <source>
```

Where `<source>` can be:

- `push`
- `pull_request`
- `schedule`
- `workflow_dispatch`
- or custom (when triggered via API with `inputs.source`)

---

## Future plan: Azure + Google dual trigger

To improve timing accuracy and observability:

### 1. Azure Functions Timer (08:00 Taipei)

- Timer trigger CRON: `0 0 0 * * *` (00:00 UTC → 08:00 Taipei)
- Calls GitHub REST API:

```http
POST https://api.github.com/repos/<OWNER>/<REPO>/actions/workflows/azure-static-web-apps-calm-sky-0ac47e700.yml/dispatches
```

With:

```json
{ "ref": "main", "inputs": { "source": "azure" } }
```

### 2. Google Cloud Scheduler + Cloud Functions (09:00 Taipei)

- Cloud Scheduler CRON: `0 9 * * *` (Asia/Taipei)
- Cloud Function calls:

```http
POST https://api.github.com/repos/<OWNER>/<REPO>/actions/workflows/azure-static-web-apps-calm-sky-0ac47e700.yml/dispatches
```

With:

```json
{ "ref": "main", "inputs": { "source": "gcp" } }
```

### 3. Co-existence with GitHub schedule

You may keep the GitHub schedule during testing.
- You will see up to three runs per day.
- The last run overwrites `build.txt`.

After verification, you can disable the schedule block if desired.

---

## Notes

- Store tokens (`GITHUB_PAT`) securely in Azure / GCP.
- Frontend only reads `build.txt`; format changes do not break functionality.
