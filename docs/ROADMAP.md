# Roadmap

Audit of the repository as of the `claude/audit` branch. No upstream-authored file was modified: every commit in this fork's history is by the upstream author, so there are no owner changes to fix, and edits would only create merge conflicts with upstream. Items are ranked by value (security and breakage first). Line numbers refer to the files at this commit. No secrets were found in any file.

YAML: the repository contains no YAML files. The XML (`menu.xml`), the supervisord INI files and `build.sh` (`bash -n`) parse cleanly. hadolint and shellcheck were not available, so Dockerfile linting was done by reading.

## 1. Likely broken builds

1. Debian buster is end-of-life and its apt repositories have been moved to the archive (`arduinoide2/Dockerfile:7`, `freefilesync/Dockerfile:26`). `apt-get update` fails on a fresh build. Move to `debian:bookworm-slim` (check package names such as `libgtk2.0-bin`, `firefox-esr`).
2. `golang:1.14-buster` is EOL and `go get ...@v1.1.0` with `go mod init` relies on pre-1.17 behaviour (`arduinoide2/Dockerfile:1-5`, `freefilesync/Dockerfile:1-5`). Use a current Go image and `go install github.com/geek1011/easy-novnc@v1.1.0`.
3. `nextcloud-ffmpeg/Dockerfile:66` hardcodes the PHP extension directory `no-debug-non-zts-20220829` (PHP 8.2). Newer Nextcloud images ship a different PHP and the COPY will fail. Derive it with `php-config --extension-dir` in the builder.
4. `nextcloud-ffmpeg/build.sh:30` sets `BUILDERVERSION=29` for every version including 27 and 28; the builder PHP may not match the target.

## 2. Security

1. The web UI is unauthenticated: `-SecurityTypes None` (`supervisord.conf:9` in both GUI images) combined with `easy-novnc --no-url-password` (`supervisord.conf:15`) and `EXPOSE 8080`. VNC itself is bound to localhost, but anyone who can reach port 8080 gets a full desktop (a terminal in the Arduino image). Add a password or put the container behind an authenticating reverse proxy, and document it.
2. `arduinoide2/Dockerfile:35` runs `chmod 0666 /dev/tty*` at start, making serial devices world-writable inside the container, and the Arduino IDE runs with `--no-sandbox` (`supervisord.conf:29`).
3. Downloads are not verified: the Arduino AppImage (`arduinoide2/Dockerfile:19`), FreeFileSync (`freefilesync/Dockerfile:13`), dlib and PDlib tarballs, and the test-suite `git clone` (`nextcloud-ffmpeg/Dockerfile:12,24,40`). Pin versions or commits and check sha256 sums.
4. `PDLIB_BRANCH=master` (`nextcloud-ffmpeg/Dockerfile:23`) is a moving target; pin a tag.
5. Containers run supervisord as an unprivileged user, which is good; the GUI images still start as root to `chown` and `usermod`. Acceptable, but worth documenting.

## 3. Reproducibility and tags

1. `build.sh:21-22` pushes an untagged (`latest`) image. Prefer explicit version tags only.
2. Base images are unpinned or pinned only by tag (`ubuntu:22.04`, `nextcloud:$VERSION`). Consider digests.
3. `pip3 install pyserial` is unpinned (`arduinoide2/Dockerfile:23`).
4. `ARG VERSION=$VERSION` (`nextcloud-ffmpeg/Dockerfile:1-2`) defaults to an empty value; the build fails confusingly if the argument is omitted. Fail early with a clear message.

## 4. Operations

1. No `HEALTHCHECK` in any image. For the GUI images, `curl -f http://localhost:8080/` is enough; for Nextcloud, `php occ status` or an HTTP probe.
2. `build.sh` has no `set -euo pipefail`, uses unquoted variables, and pushes to the upstream author's Docker Hub namespace (`kilrah/`). Parameterise the namespace before using it from this fork.
3. `make -j` without a job limit (`nextcloud-ffmpeg/Dockerfile:19,30`) can exhaust memory on small builders; use `make -j"$(nproc)"`.
4. `rm -rf /var/lib/apt/lists` removes the directory rather than its contents (`arduinoide2/Dockerfile:14`, `freefilesync/Dockerfile:32`); use `/var/lib/apt/lists/*`.
5. Trailing blank lines and trailing whitespace at the end of both GUI Dockerfiles and after `\` on `arduinoide2/Dockerfile:22`.

## 5. Housekeeping

1. Decide whether this fork should track upstream (keep files untouched, sync regularly) or diverge (then apply the items above).
2. Add CI: hadolint, shellcheck, and a build smoke test.
3. Add per-image READMEs (ports, volumes, environment variables such as `PUID` and `PGID`).
4. Add the Unraid XML templates the README refers to, or link them explicitly.
