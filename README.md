# Project P

A personal manga library.

[Latest release](https://github.com/pproject-labs/project-p.release/releases/latest) ·
[All releases](https://github.com/pproject-labs/project-p.release/releases) ·
[Report a bug](https://github.com/pproject-labs/project-p.release/issues/new)

Project P is a self-hosted manga library. Use the desktop app on one computer, or run the
server on a NAS or home server and open it from any browser.

These are early builds. Packages are unsigned.

> **Release status:** the first Latest release after the channel update may still be pending.
> If the Latest link has no files yet, use the newest Nightly build. The installer and channel
> layout described below starts with that first Latest release.

## Choose your install

| I want to... | Recommended |
| --- | --- |
| Use the desktop app on this computer | macOS Apple Silicon DMG, Windows x64 Setup, or Linux x64 AppImage |
| Run a server on a NAS or home server | `ghcr.io/pproject-labs/project-p:latest` |
| Run a standalone server on Linux or macOS | `install.sh` from the latest release |
| Run a standalone server on Windows | `install.ps1` from the latest release |

**Latest** is the default channel. **Nightly** builds are for testing; they may change data
formats and cannot always be rolled back.

## Desktop app

Download the file for your computer from the [latest release](https://github.com/pproject-labs/project-p.release/releases/latest):

| Platform | File |
| --- | --- |
| macOS Apple Silicon | `Project-P-<version>-macOS-arm64.dmg` |
| Windows x64 | `Project-P-<version>-Windows-x64-Setup.exe` |
| Linux x64 | `Project-P-<version>-Linux-x64.AppImage` |

The macOS build is Apple Silicon only. Windows and Linux desktop packages are x64.
Unsigned macOS builds may require the system approval flow; Windows may show a SmartScreen
warning. Verify the download against `SHA256SUMS` before opening it.

## Docker

Use the published image:

```sh
docker run -d --name project-p \
  --restart unless-stopped \
  -p 127.0.0.1:8800:8800 \
  -v project-p-data:/data \
  ghcr.io/pproject-labs/project-p:latest
```

Or save this as `compose.yaml`:

```yaml
services:
  project-p:
    image: ghcr.io/pproject-labs/project-p:latest
    container_name: project-p
    restart: unless-stopped
    ports:
      - "127.0.0.1:8800:8800"
    environment:
      PROJECT_P_BIND: 0.0.0.0:8800
    volumes:
      - ./data:/data
      # Add existing Manga files here and use /manga as the Library Path:
      # - ./manga:/manga
```

Start it with `docker compose up -d`, then read the startup output:

```sh
docker compose logs -f project-p
```

If you used `docker run`, replace `docker compose logs` with `docker logs` and
`docker compose exec` with `docker exec` in the commands below.

The container binds `0.0.0.0:8800` internally, so Docker requests need a Pairing Code even
when the published host port is loopback-only. Open `http://localhost:8800/pair` and enter
the code from the logs, or request a fresh link:

```sh
docker compose exec project-p project-p pair --public-url http://localhost:8800
```

For LAN access, publish `8800:8800` and pair each browser from the logs. Use a direct HTTP
connection only on a trusted private network; put a TLS reverse proxy in front of an
internet-facing instance.

The `/data` volume holds application state. To use existing Manga files, mount their host
directory into the container and enter the **container path** when adding a Library Path. The
container runs as UID `10001`, so the mounted directory needs the permissions required for
the operations you enable.

## Standalone server

On Linux x64 or macOS Apple Silicon, install the executable into `~/.local/bin`:

```sh
curl --fail --silent --show-error --location \
  https://github.com/pproject-labs/project-p.release/releases/latest/download/install.sh \
  --output install.sh
sh install.sh
project-p serve --open
```

On Windows x64, use PowerShell:

```powershell
irm https://github.com/pproject-labs/project-p.release/releases/latest/download/install.ps1 | iex
project-p serve --open
```

Both installers download the platform archive, verify it against the release `SHA256SUMS`, and
install without requiring elevated privileges by default. Use `--dry-run` to inspect the
planned installation, or `--version <version>` to install a specific release.

Linux arm64 is available through the Docker image; a native Linux arm64 archive is not
published. macOS Intel is not published.

## Verify downloads

Every release includes one `SHA256SUMS` file. Compare the hash of the file you downloaded:

```sh
sha256sum project-p-<version>-linux-x64.tar.gz   # Linux
shasum -a 256 project-p-<version>-macos-arm64.tar.gz  # macOS
```

The matching line in `SHA256SUMS` must have the same hash. The installers perform this check
automatically.

## Upgrade and back up

Stop the running server or container and back up the data directory before replacing a build.
Database migrations may not be reversible. Docker data lives in the `project-p-data` volume;
standalone data defaults to `~/.project-p`. Manga files in separate Library Paths are not part
of that data directory and need their own backup.

## Development

Project P is under active development. This repository publishes builds and collects bug
reports; the development source is private. Use the issue tracker for reproducible bugs and
include the Project P version, platform, and relevant logs with secrets and pairing codes
removed.
