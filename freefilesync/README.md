# freefilesync (FreeFileSync over noVNC)

FreeFileSync 13.6 desktop in a browser, using TigerVNC + easy-novnc + Openbox.

| Item | Value |
|---|---|
| Web UI | `http://<host>:8080` (noVNC). **No authentication** — publish only on a trusted LAN or behind an authenticating reverse proxy. |
| VNC | 5900, bound to `localhost` inside the container only (not reachable from outside). |
| Runs as | UID 99 / GID 100 (Unraid `nobody:users`), fixed at build time. |
| Healthcheck | TCP connect to `127.0.0.1:8080`. |

## Volumes

| Container path | Purpose |
|---|---|
| `/home/freefilesync/path1/` | Sync side 1 (default left folder) |
| `/home/freefilesync/path2/` | Sync side 2 (default right folder) |
| `/home/freefilesync/.config/FreeFileSync/` | App settings/batch jobs (also reachable as `~/appdata`) |

Host directories must be readable/writable by UID 99 / GID 100, otherwise FreeFileSync cannot read or write them.

## Build

```bash
docker build -t freefilesync-vnc freefilesync/
```

Restart policy, port publishing and resource limits are set by the Unraid template / `docker run`, not by this Dockerfile. See `docs/docker-audit.md` for known issues.
