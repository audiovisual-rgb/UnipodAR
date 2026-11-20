# Connecting Unipod AR to Git

The project is **already a git repository** with a clean first commit — `main` branch, nothing uncommitted.
What's left is attaching a remote (GitHub / GitLab / Bitbucket) and, optionally, letting Vercel redeploy on push.

```
unipod-ar/
├─ index.html              # the app (source of truth, single file)
├─ api/health.js           # serverless backend: GET /api/health
├─ vercel.json             # static config + camera Permissions-Policy
├─ package.json            # build / dev / deploy scripts
├─ scripts/build.mjs       # esbuild → dist/ (minified)
├─ scripts/dev.mjs         # local server that emulates Vercel (static + /api/health)
├─ .github/workflows/      # optional: deploy + hourly smoke test
├─ .gitignore  .gitattributes  .vercelignore
└─ README.md  DEPLOY.md  VERIFICATION.md
```

Current history:

```
8cfd199  Unipod AR: AR scanner dashboard (sky blue + yellow), static app, /api/health backend, Vercel config
```

---

## 1. Push to GitHub

**With the GitHub CLI** (fastest — creates the repo *and* pushes):

```bash
cd unipod-ar
gh auth login                    # once
gh repo create unipod-ar --private --source=. --remote=origin --push
```

**With plain git** (create the empty repo on github.com first, don't add a README):

```bash
cd unipod-ar
git remote add origin https://github.com/<you>/unipod-ar.git
git push -u origin main
```

## 2. Push to GitLab or Bitbucket

```bash
# GitLab  (create an empty project first)
git remote add origin https://gitlab.com/<you>/unipod-ar.git
git push -u origin main

# Bitbucket (create an empty repository first)
git remote add origin https://bitbucket.org/<you>/unipod-ar.git
git push -u origin main
```

## 3. Connect the repo to Vercel (auto-deploy on every push)

1. [vercel.com/new](https://vercel.com/new) → **Import Git Repository** → authorise GitHub/GitLab/Bitbucket → pick `unipod-ar`.
2. Project settings: **Framework Preset = Other**, **Build Command = `npm run build`** (or leave empty to serve the
   single-file app), **Output Directory = `.`**.
3. **Deploy.** Vercel then redeploys automatically on every push to `main`, and `GET /api/health` goes live with it.

> The repo ships `vercel.json`, so the camera header (`Permissions-Policy: camera=(self)`), clean URLs and cache rules
> are applied without touching the dashboard.

## 4. Optional — GitHub Actions deploys instead of the Vercel Git integration

`.github/workflows/deploy.yml` builds and runs `vercel deploy --prod` on every push to `main`.
Add three repository secrets (Settings → Secrets and variables → Actions):

| Secret | Where to get it |
|---|---|
| `VERCEL_TOKEN` | vercel.com/account/tokens → Create |
| `VERCEL_ORG_ID` | Vercel project → Settings → General → Team ID |
| `VERCEL_PROJECT_ID` | Vercel project → Settings → General → Project ID |

`.github/workflows/healthcheck.yml` is a scheduled smoke test: set a repository **variable**
`UNIPOD_SITE_URL=https://<project>.vercel.app` and it curls `/api/health` hourly.

## 5. Authentication notes

- Arena has **no GitHub/GitLab connector** available, so the push itself needs your credentials — either run the
  commands above locally, or paste a **personal access token** here and the push can be done from this workspace
  (`https://<token>@github.com/<you>/unipod-ar.git`).
- Never commit tokens. `.gitignore` already excludes `.vercel/`, `node_modules/` and `dist/`.
- Local commit identity is currently `Unipod AR <dev@unipod.ar>`; change it with
  `git config user.name "Your Name" && git config user.email "you@example.com"` and
  `git commit --amend --reset-author`.
