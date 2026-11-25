# ASCMS

ASCMS is a self-hosted CMS built with NestJS 11 and Vue 3. It ships with an installation wizard, content/category/model management, custom forms, a template engine, backup & restore, and online upgrades.

This is the **official release repository**. It only contains release artifacts and documentation, not source code.

- Website: <https://ascms.cmsuse.com>
- Live demo: <https://demo.cmsuse.com>
- Latest release: [Gitee Releases](https://gitee.com/gas0324/ascms-releases/releases/latest)
- GitHub mirror: <https://github.com/Gas0515/ascms-releases>

## Downloads

Open the latest Release and download an attachment:

| Asset | Purpose |
| --- | --- |
| `ascms-<version>.zip` | **Recommended.** Complete package (backend `dist`, built admin frontend, `init.sql`). Used for manual installs and, from 1.0.3, for online upgrades |
| `program-<version>.zip` | Legacy online-upgrade package. Upgraders in 1.0.0–1.0.2 only recognize this prefix. 1.0.3 provides a compatibility copy; 1.0.4+ no longer publishes it |
| `upgrade-<version>.sql` | Incremental database script, only present when the release changes the schema. The upgrader runs it automatically |

> ⚠️ The auto-generated `v<version>.zip` / `v<version>.tar.gz` files on the Release page are source snapshots (this repository only contains documentation). They **cannot be deployed or used for upgrades**. Download only the assets listed above.

## Online upgrade (recommended)

1. Sign in to the admin panel and open **System → System Upgrade**
2. Click **Check for updates**, then **Upgrade now**
3. The system downloads the package, backs up the database, runs incremental SQL, replaces program files, and restarts

Only `dist`, `package.json`, and `frontend` are replaced. `.env`, `.installed`, `data`, `logs`, `backup`, `backend/public`, and `backend/sql` are preserved. Private templates are backed up before replacement and restored afterwards.

Notes:

- The current version is read from `as_system.version` (since 1.0.2), then `data/version`, then the packaged `package.json`
- A full database backup is taken before every upgrade; rollback of schema changes relies on that backup
- Upgrade versions in order. The upgrader collects every SQL attachment in the version range
- **Docker deployments**: official image `gas0324/ascms` ([Docker Hub](https://hub.docker.com/r/gas0324/ascms)). `/app/backend` and `/app/frontend` are persisted as named volumes, so recreating the container keeps both code and configuration. Upgrade from **System → System Upgrade**; do not replace the image to upgrade

## Manual installation

Requirements:

| Item | Requirement |
| --- | --- |
| Node.js | 20 LTS or newer |
| MySQL | 5.7 or newer (8.0 recommended) |
| OS | Linux x64 / Windows |
| Memory | 1 GB or more recommended |
| Disk | 2 GB or more recommended |

```bash
# 1. Extract
unzip ascms-<version>.zip -d /www/ascms
cd /www/ascms/backend

# 2. Install production dependencies
npm install --omit=dev

# 3. Start
NODE_ENV=production node dist/main.js
```

The app listens on port `9000` by default (`PORT` overrides it). On first start, `.installed` is missing, so open `http://<host>:9000/install` and complete the wizard with your database and site settings. The schema is created automatically. The admin URL is the `FRONTEND_ROUTE` value in `.env`.

For production, run the process under PM2 or systemd and put Nginx/OpenResty in front of it (`client_max_body_size 50m`; WebSocket `/socket.io/` is required for upgrade progress).

> Full Docker, reverse proxy, and HTTPS instructions: <https://ascms.cmsuse.com/deploy>

## Version history

| Version | Date | Highlights |
| --- | --- | --- |
| [1.0.8](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.8) | 2026-11-24 | Fix 404 on static content pages; record real client IP and detect internal referrers by request host |
| [1.0.7](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.7) | 2026-11-24 | Mail settings: SMTP host/port/encryption/account/password, connection test and test email; sender address always follows the login account; passwords never echoed and blank saves keep the old value; 9 new `as_system` columns (incremental SQL) |
| [1.0.6](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.6) | 2026-11-23 | Static generation skips categories without templates; static/remote mode 302-redirects the home page to `index.html` and returns 404 for missing static pages; redesigned upgrade dialog with download speed and ETA |
| [1.0.5](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.5) | 2026-11-22 | Fixed static generation (default URL rules, empty categories, content pages for link-type categories); website run mode now takes effect (dynamic/static/remote switch the link scheme, missing static files fall back to on-the-fly rendering) |
| [1.0.4](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.4) | 2026-11-22 | Fixed custom-template static assets (e.g. `/yujiekt`) being swallowed by the frontend 404 fallback; redesigned upgrade progress modal; from 1.0.4 online upgrades ship `ascms-` only |
| [1.0.3](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.3) | 2026-11-21 | Product site merged into the built-in default template (light theme); Gzip toggle + static long-cache; fixed search XSS, frontend 404, install wizard; upgrade packages now use the `ascms-` prefix (with a `program-` compat copy) |
| [1.0.2](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.2) | 2026-11-18 | Fixed online upgrade (volume EBUSY, progress stream, version tracking), password saving, visit statistics, and admin top bar |
| [1.0.1](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.1) | 2026-11-18 | Fixed upgrade dependency, duplicate labels, login autofill, and dashboard cards |
| [1.0.0](https://gitee.com/gas0324/ascms-releases/releases/tag/v1.0.0) | 2026-11-17 | First stable release: install wizard, content/category/forms, backup & restore, online upgrade, templates |

See each Release for the full changelog.

## Feedback

- Issues: <https://gitee.com/gas0324/ascms-releases/issues>
- Contact: <https://ascms.cmsuse.com/contact>

---

© ASCMS. This repository is used for official release distribution only.
