# Project P

A personal manga library.

[Latest release](https://github.com/pproject-labs/project-p.release/releases/latest) ·
[All releases](https://github.com/pproject-labs/project-p.release/releases) ·
[Report a bug](https://github.com/pproject-labs/project-p.release/issues/new)

Project P runs a manga library on your computer or home server. Use the desktop app, or open
the server from any browser. Your files stay on the machine you control.

These are early builds. Packages are unsigned.

> **Release status:** the first Latest release after the channel update may still be pending.
> If the Latest link has no files yet, use the newest Nightly build.

## Getting started

### Desktop

Download the file for your computer from the [latest release](https://github.com/pproject-labs/project-p.release/releases/latest).
The app starts its own local server.

| Platform            | File                                        |
| ------------------- | ------------------------------------------- |
| macOS Apple Silicon | `Project-P-<version>-macOS-arm64.dmg`       |
| Windows x64         | `Project-P-<version>-Windows-x64-Setup.exe` |
| Linux x64           | `Project-P-<version>-Linux-x64.AppImage`    |

The macOS build is Apple Silicon only. macOS and Windows may ask you to approve an unsigned app.

### Docker

```sh
docker run -d --name project-p \
  --restart unless-stopped \
  -p 127.0.0.1:8800:8800 \
  -v project-p-data:/data \
  ghcr.io/pproject-labs/project-p:latest
```

Open `http://localhost:8800`. Docker requests need a pairing code even on a loopback host; read
it with `docker logs -f project-p`, or print a fresh link:

```sh
docker exec project-p project-p pair --public-url http://localhost:8800
```

Or save this as `compose.yaml` and run `docker compose up -d`:

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

### Standalone server

Linux x64 and macOS Apple Silicon:

```sh
curl -fsSL https://github.com/pproject-labs/project-p.release/releases/latest/download/install.sh | sh
project-p serve --open
```

Windows x64:

```powershell
irm https://github.com/pproject-labs/project-p.release/releases/latest/download/install.ps1 | iex
project-p serve --open
```

The installers verify the archive against the release `SHA256SUMS` and install without
elevation. Use `--version <version>` to install a specific release. Linux arm64 is available
through Docker; macOS Intel is not published.

## First library

Create a library in the app, or after the server is running:

```sh
project-p login --local
project-p library init "$HOME/Manga" --name "Manga"
```

For Docker, run the same commands inside the container:

```sh
docker exec project-p project-p login --local
docker exec project-p project-p library init /data/library --name "Manga"
```

Use `--ownership referenced` for a folder Project P must keep read-only. To use existing Manga
files in Docker, mount the host folder and enter the **container path** (`/manga`) when adding
the library. The container runs as UID `10001`.

## CLI

`project-p --help` lists the commands. The desktop app can install the same CLI from
**Settings → About**. The CLI can point at a remote server; `login` remembers the server for
later commands:

```sh
project-p login 'https://your-host/pair#token=<pairing-token>'
project-p api library.list --json
```

Every Control Plane operation is available through `project-p api`, including create, read,
update, and delete for libraries, manga, collections, subscriptions, reading state, downloads,
backups, and settings. `project-p openapi` prints the contract.

## Upgrades and backups

Stop the server or container before replacing a build, and back up the data directory.
Standalone data defaults to `~/.project-p`; Docker data lives in the `project-p-data` volume.
Manga files in separate Library Paths need their own backup. Database migrations may not be
reversible.

## Verify downloads

Every release includes one `SHA256SUMS` file. Compare the hash of the file you downloaded with
its line in `SHA256SUMS`:

```sh
sha256sum project-p-<version>-linux-x64.tar.gz      # Linux
shasum -a 256 project-p-<version>-macos-arm64.tar.gz # macOS
```

The installers perform this check automatically.

## Notes

- Pairing codes are single-use, expire after five minutes, and grant Control Plane access.
- Put a TLS reverse proxy in front of an internet-facing server; do not expose the HTTP
  listener directly.
- **Latest** is the default channel. **Nightly** builds are for testing and may change data
  formats.
- Project P is under active development. This repository publishes builds and collects bug
  reports; the development source is private.
