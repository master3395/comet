# Stream deployment updater

This document describes how the Comet admin dashboard **System** tab performs backup, upgrade, and downgrade on stream.newstargeted.com-style hosts.

## Overview

The System tab UI lives in Comet (`comet/templates/admin_dashboard.html`). Host-level Docker operations are handled by a PHP module beside the vhost docroot, exposed at `/operator/api/*`.

```text
Admin dashboard (Comet, /admin/dashboard)
  -> fetch /operator/api/status|action (credentials: same-origin)
  -> PHP validates admin_session cookie OR Discord operator session
  -> sudo comet-{backup,update,restore}.sh
```

## Requirements

1. **LiteSpeed vhost**: `/operator` must be served by PHP **before** the catch-all Comet proxy. See the production `vhost.conf` context for `/operator`.
2. **Sudoers**: `/etc/sudoers.d/stream-comet-updater` allowing the web user to run the updater shell scripts.
3. **Password sync**: `ADMIN_DASHBOARD_PASSWORD` in Comet `deployment/.env` must match what PHP reads from the same file (used to verify the `admin_session` cookie).
4. **Backups**: written under `/home/backup/` as `stream-comet-pre-{tag}-{timestamp}/`.

## Authentication

| Method | Used by | Notes |
|--------|---------|-------|
| Comet admin password | `/admin/dashboard` System tab | Signed `admin_session` cookie (path `/`) |
| Discord OAuth | `/operator/` fallback panel | Allowed Discord user IDs in PHP `config.php` |

Both methods can call `/operator/api/status` and `/operator/api/action`.

## PHP module layout (production)

On stream.newstargeted.com the live tree is:

```text
stream.newstargeted.com/
  config.php
  modules/updater/lib/
    auth.php
    comet_admin_auth.php
    dashboard.php
  modules/updater/bin/
    comet-backup.sh
    comet-update.sh
    comet-restore.sh
  operator/api/
    status.php
    action.php
```

Copy or sync this tree when deploying on a new host.

## Update workflow

1. Sign in at `/admin` with the dashboard password.
2. Open **System** tab.
3. Choose Docker tag (`main`, `latest`, or a release tag).
4. Click **Update now** (backup runs first; only the three newest backups are kept).
5. To downgrade, use **Restore** on a backup row.

The Discord `/operator/` panel remains available as a fallback.

## GitHub repo for update checks

The System tab **Update available** check uses [g0ldyy/comet](https://github.com/g0ldyy/comet) by default. Override with `COMET_GITHUB_REPO` if you track a different fork.

## Related docs

See the server `to-do/OPERATOR-UPDATER.md` on stream.newstargeted.com for Discord OAuth setup, sudoers validation, and manual CLI commands.
