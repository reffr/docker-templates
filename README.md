# docker-templates

Source repo for my [unraid templates](https://github.com/kilrah/unraid-docker-templates)
## Repository structure

This repository is a fork of [kilrah/docker-templates](https://github.com/kilrah/docker-templates). Each top-level directory is one image and is self-contained:

| Directory | Purpose | Contents |
|---|---|---|
| `arduinoide2/` | Arduino IDE 2 in a browser (noVNC) | `Dockerfile`, `supervisord.conf`, `menu.xml` |
| `freefilesync/` | FreeFileSync GUI in a browser (noVNC) | `Dockerfile`, `supervisord.conf`, `menu.xml` |
| `nextcloud-ffmpeg/` | Nextcloud with ffmpeg, VA-API drivers and PDlib | `Dockerfile`, `build.sh` |

### Conventions

- The two GUI images share one pattern: Xtigervnc on localhost:5900, `easy-novnc` serving the web UI on port 8080, openbox as window manager, all run by `supervisord`. `menu.xml` is the openbox right-click menu.
- The container user is uid 99 / gid 100 (Unraid `nobody:users`).
- Persistent data lives in `VOLUME` paths; map them to appdata shares.
- `nextcloud-ffmpeg/build.sh` builds and pushes images per Nextcloud version. It currently targets the upstream author's Docker Hub namespace; change it before running from this fork.
- Changes to upstream-authored files are intentionally avoided in this fork so it stays easy to sync with upstream. Findings from an audit are recorded in [docs/ROADMAP.md](docs/ROADMAP.md) rather than applied.
