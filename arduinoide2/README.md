# arduinoide2 (Arduino IDE 2 over noVNC)

Arduino IDE 2.3.2 (AppImage) plus terminal, file manager, Geany and Firefox, in a browser via TigerVNC + easy-novnc + Openbox.

| Item | Value |
|---|---|
| Web UI | `http://<host>:8080` (noVNC). **No authentication**, and the desktop includes a shell — publish only on a trusted LAN or behind an authenticating reverse proxy. |
| VNC | 5900, bound to `localhost` inside the container only. |
| Healthcheck | TCP connect to `127.0.0.1:8080`. |

## Environment

| Variable | Default | Purpose |
|---|---|---|
| `PUID` | `99` | UID the `arduino` user is remapped to at start |
| `PGID` | `100` | GID the `users` group is remapped to at start |

## Volumes

| Container path | Purpose |
|---|---|
| `/home/arduino/` | Sketches, IDE settings, board cores and libraries |

On every start the entrypoint runs `chown -R $PUID:$PGID /home/arduino`, which can be slow on large volumes and changes ownership of everything in the mapped folder.

## Serial devices

Pass boards through with `--device /dev/ttyUSB0` (or `/dev/ttyACM0`). The entrypoint makes `/dev/tty*` world-writable inside the container so the unprivileged user can open them.

## Build

```bash
docker build -t arduinoide2-vnc arduinoide2/
```

See `docs/docker-audit.md` for known issues.
