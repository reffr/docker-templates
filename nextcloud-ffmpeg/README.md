# nextcloud-ffmpeg

Official `nextcloud` image plus:

- `ffmpeg` and VA-API drivers (Intel `intel-media-va-driver-non-free`, `i965-va-driver`, AMD/`mesa-va-drivers`, `vainfo`) for Memories/preview transcoding with hardware acceleration;
- dlib + the `pdlib` PHP extension for the Face Recognition app;
- the `bz2` PHP extension, `nano`, `ncdu`.

Ports, volumes, environment variables and healthcheck behaviour are those of the upstream [`nextcloud`](https://hub.docker.com/_/nextcloud) image (apache variant: port 80; `-fpm` variant: port 9000). For hardware transcoding pass `/dev/dri` to the container.

## Build arguments

| Arg | Purpose |
|---|---|
| `VERSION` | Tag of the final `nextcloud` base image, e.g. `29.0.2` or `29.0.2-fpm`. **Required** (no usable default). |
| `BUILDERVERSION` | Tag of the `nextcloud` image used to compile dlib/pdlib, e.g. `29`. **Required**. Must use the same PHP version as `VERSION` (the extension is copied from `no-debug-non-zts-20220829`, i.e. PHP 8.2). |
| `DLIB_BRANCH` | dlib git ref (default `v19.24`). |
| `PDLIB_BRANCH` | pdlib git ref (default `master`). |

`build.sh` builds and **pushes** all tags to `kilrah/nextcloud-ffmpeg` (needs `docker login`).

See `docs/docker-audit.md` for known issues.
