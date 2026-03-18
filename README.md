# GitHub Pages Visibility POC

## Purpose
Test whether GitHub Pages (from a private repo in the `Green-Dot-Corporation` org) restricts access to org members only.

## Quick Start

### Option A: Local Test (before pushing)
```powershell
# From this directory:
.\test-local.ps1
```
Opens the page in your browser at `http://localhost:8080` — verifies the page works before deploying.

### Option B: Deploy to GitHub Pages
1. Create a **private** repo: `Green-Dot-Corporation/gh-pages-visibility-poc`
2. Push this folder's contents
3. Go to repo Settings → Pages → Source: GitHub Actions
4. The workflow auto-deploys on push to `main`
5. URL: `https://green-dot-corporation.github.io/gh-pages-visibility-poc/`

## Visibility Test Scenarios

| # | Scenario | How to Test | Expected (Private Repo) | Expected (Public Repo) |
|---|----------|-------------|------------------------|----------------------|
| 1 | GD employee, logged into GitHub, on VPN | Open URL normally | ✅ Visible | ✅ Visible |
| 2 | GD employee, logged into GitHub, OFF VPN | Disconnect VPN, open URL | ✅ Visible (Enterprise) | ✅ Visible |
| 3 | GD employee, NOT logged into GitHub | Incognito browser | ❌ 404 | ✅ Visible |
| 4 | External person (not in GD org) | Ask someone outside GD | ❌ 404 | ✅ Visible |
| 5 | No GitHub account | Private/incognito, clear cookies | ❌ 404 | ✅ Visible |
| 6 | Google / search engine crawler | Check robots.txt | ❌ Blocked by robots.txt + private | ❌ Blocked by robots.txt |
| 7 | Client-side Jira API call | Page auto-tests this | ❌ CORS blocked | ❌ CORS blocked |
| 8 | Pre-baked data freshness | Page auto-tests this | ✅ Shows embedded data | ✅ Shows embedded data |

## Key Decision: Private vs Public Repo

### Private Repo (Recommended for internal tools)
- **Access**: Only GitHub org members can view Pages
- **Requires**: GitHub Enterprise Cloud with Pages enabled for private repos
- **Verify**: Repo Settings → Pages → should show "Access control" section
- **Security**: No data leakage even if someone guesses the URL

### Public Repo (Acceptable if no sensitive data in HTML)
- **Access**: Anyone can view
- **Acceptable when**: All sensitive data is fetched server-side (GitHub Actions) and only aggregated/non-sensitive metrics are baked in
- **Risk**: Anyone can see team metrics, sprint counts, etc.

## Architecture

```
GitHub Actions (scheduled cron)
    │
    ├─ Fetch Jira API (cloud, no VPN needed)
    ├─ Run build script (bake data into HTML)
    └─ Deploy static HTML to GitHub Pages
           │
           └─ Users visit URL → see pre-built dashboard
              (no API calls at runtime, no VPN needed)
```

## Files
- `docs/index.html` — Visibility test page with automated probes
- `docs/robots.txt` — Blocks search engine indexing
- `.github/workflows/deploy-pages.yml` — GitHub Actions deployment
- `test-local.ps1` — Local testing script
- `test-scenarios.ps1` — Automated scenario validation
