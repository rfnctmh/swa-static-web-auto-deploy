# Azure Static Web Apps Demo (auto deploy + last build time)

This project demonstrates:
- Auto deploy to Azure Static Web Apps on every push
- Daily scheduled redeploy
- Frontend shows the latest build time from `build.txt`

## Usage
1. Push this repo to your GitHub account
2. Create Azure Static Web App and link to this repo (app location = `site`)
3. In GitHub → Settings → Secrets → Actions, add:
   - `AZURE_STATIC_WEB_APPS_API_TOKEN`
4. After each push or at 00:00 UTC daily, GitHub Actions will:
   - write `site/build.txt` with current UTC time
   - deploy to SWA
5. The page will fetch `build.txt` and display "Last build: ...".
