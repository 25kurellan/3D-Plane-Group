# AGENTS.md

## What this project is

An F-Droid repository template (GAF-Droid) repurposed as a small **PWA app store** page.

- `index.html` at the repo root is the entire runnable app: one self-contained page with
  inline CSS/JS, no build step, no backend and no network calls.
  - Auth, users, catalog apps and per-user overrides live in `localStorage`
    (`db_u` / `db_a`), seeded on first load from constants in the page script.
  - The seeded admin account is defined in the page (`a-person2@outlook.com`). Sign in
    with the seeded admin to see the store grid plus the "Admin Control Engine" panel;
    any other role only sees the grid.
- `fdroid/repo/config.yml` + `metadata/` describe the F-Droid repository. It is served
  as static files but holds no signed index yet — the index is produced in CI.
- `metascoop/` is a **Go CLI used only by CI** (`update.sh`, called from the GitHub
  workflow). It clones each app repo, collects releases/screenshots and regenerates the
  F-Droid index and the README table. It is not part of the preview.

## Running it

```
docker compose -f docker-compose.base44.yml up -d
```

`web` is nginx:alpine serving the repo root from a bind mount on host port 3000
(`Cache-Control: no-store`, so edits to `index.html` appear on refresh). The nginx
config runs workers as root because the sandbox's bind-mounted source directories are
not world-readable; the config lives in `.base44/nginx.conf`.

## Quirks

- The app is 100% client-side: "state" changes (new apps, users) are only in the
  browser's `localStorage` and do not touch the repo. A hard reload after clearing
  storage restores the seeds.
- `metascoop` needs `GH_ACCESS_TOKEN` (a GitHub token with no scopes) plus the
  `CONFIG_YML` / `KEYSTORE_P12` CI secrets to regenerate the repository. Without them
  `./update.sh` fails, and that does not affect the preview site.
- `apps.yaml` maps app ids to GitHub repos; its entries drive what CI imports, not what
  the page displays.

## Verifying

- `curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:3000/` → `200`.
- In the preview: the "Gate Login" card shows first; signing in with the seeded admin
  reveals the "Directory" grid with a "3D Plane Group" card and the admin panel.
