# Project P

A personal manga library.

[Latest release](https://github.com/pproject-labs/project-p.release/releases/latest) ·
[All releases](https://github.com/pproject-labs/project-p.release/releases) ·
[Report a bug](https://github.com/pproject-labs/project-p.release/issues/new)

Project P is a self-hosted manga library. Use the desktop app on one computer, or run the
server on a NAS or home server and open it from any browser. Storage, scanning, downloads, and
metadata stay on the machine you control.

These are early builds. Packages are unsigned.

[Getting started](#getting-started) · [Agent-assisted setup](#agent-assisted-setup) ·
[CLI](#cli) · [Backup and upgrade](#data-backup-and-upgrade) · [Troubleshooting](#troubleshooting)

> **Release status:** the first Latest release after the channel update may still be pending.
> If the Latest link has no files yet, use the newest Nightly build. The installer and channel
> layout described below starts with that first Latest release.

## What it does

- Browse, search, and read a local Manga Library in the desktop app or a browser.
- Scan CBZ/CBR/ZIP archives and keep reading progress in sync across devices.
- Search built-in Source Extensions, import candidates, and download Chapters.
- Follow updates and subscriptions, with background jobs visible in the app.
- Create portable backups and restore them without moving your original archives.
- Automate the catalog through the CLI and the full OpenAPI Contract.

## Quick start

| I want to...                              | Use                                                               |
| ----------------------------------------- | ----------------------------------------------------------------- |
| Use the desktop app on this computer      | macOS Apple Silicon DMG, Windows x64 Setup, or Linux x64 AppImage |
| Run a server on a NAS or home server      | `ghcr.io/pproject-labs/project-p:latest`                          |
| Run a standalone server on Linux or macOS | `install.sh` from the latest release                              |
| Run a standalone server on Windows        | `install.ps1` from the latest release                             |

**Latest** is the default channel. **Nightly** builds are for testing; they may change data
formats and cannot always be rolled back.

## Getting started

### Docker

Docker is the recommended path for a NAS or an always-on home server.

```sh
docker run -d --name project-p \
  --restart unless-stopped \
  -p 127.0.0.1:8800:8800 \
  -v project-p-data:/data \
  ghcr.io/pproject-labs/project-p:latest
```

Open `http://localhost:8800`. The container binds `0.0.0.0:8800` internally, so Docker
requests need a Pairing Code even when the published host port is loopback-only. Read the
startup output:

```sh
docker logs -f project-p
```

Open `http://localhost:8800/pair` and enter the code, or request a fresh link:

```sh
docker exec project-p project-p pair --public-url http://localhost:8800
```

If the standalone CLI runs on the host, `docker exec project-p project-p pair` prints a code
that can be exchanged with
`project-p --api-url http://localhost:8800/api login <code>`.

To use existing Manga files, mount their host directory and enter the **container path** when
adding a Library Path:

```sh
docker run -d --name project-p \
  --restart unless-stopped \
  -p 127.0.0.1:8800:8800 \
  -v project-p-data:/data \
  -v /path/to/manga:/manga \
  ghcr.io/pproject-labs/project-p:latest
```

Use `/manga` as the Library Path inside Project P. The container runs as UID `10001`, so a
mounted directory needs the permissions required for the operations you enable. Use a
`referenced` Library Path for read-only files, or make the mount writable for downloads and
organizing. For a fresh install, `/data/library` is writable without host permission setup.

For LAN access, publish `8800:8800` and pair each browser from the logs. Use a direct HTTP
connection only on a trusted private network; put a TLS reverse proxy in front of an
internet-facing instance.

#### Docker Compose

Save this as `compose.yaml`:

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

Start it with `docker compose up -d`, then read `docker compose logs -f project-p` and follow
the same pairing steps. Replace `docker` with `docker compose` in the commands above.

### Desktop app

Download the file for your computer from the
[latest release](https://github.com/pproject-labs/project-p.release/releases/latest):

| Platform            | File                                        |
| ------------------- | ------------------------------------------- |
| macOS Apple Silicon | `Project-P-<version>-macOS-arm64.dmg`       |
| Windows x64         | `Project-P-<version>-Windows-x64-Setup.exe` |
| Linux x64           | `Project-P-<version>-Linux-x64.AppImage`    |

The macOS build is Apple Silicon only. Windows and Linux desktop packages are x64. The app
starts its own local server and opens the Web client; no separate `project-p serve` process is
needed. Create your first library from the app.

Unsigned macOS builds may require the system approval flow; Windows may show a SmartScreen
warning. Verify the download against `SHA256SUMS` before opening it.

### Standalone server

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

## Create your first library

A **Library Path** is a folder on the machine running Project P. Create the first library in
the Web app, or with the CLI after the server is running.

For a standalone server:

```sh
project-p login --local
project-p library init "$HOME/Manga" --name "Manga"
```

For Docker, run the CLI inside the container:

```sh
docker exec project-p project-p login --local
docker exec project-p project-p library init /data/library --name "Manga"
```

The CLI session created by `docker exec` lives inside the container; repeat `login --local`
after recreating the container. A host CLI paired with
`project-p --api-url http://localhost:8800/api login <code>` keeps its session on the host.

Use `--ownership referenced` when Project P must keep an existing folder read-only. The default
`managed` mode allows Project P to place downloaded and organized files there. `library init`
also starts the initial scan and waits for it by default.

The browser shows the same result. If the folder is empty, put CBZ/CBR/ZIP files there and run
`project-p library scan <library-id>`, or use the Refresh action in the Web app.

## Agent-assisted setup

The CLI is designed to be driven by an agent as well as a person. A coding agent can install
Project P, pair the CLI, create a library, scan it, search sources, import manga, download
chapters, and manage catalog state.

Paste a prompt like this into your agent:

```text
Set up Project P on this machine using the public release repository:
https://github.com/pproject-labs/project-p.release

1. Detect the OS and architecture and check whether Docker is available.
2. Ask me whether this should run on this computer or on a NAS/server, and where my manga
   files live. Do not guess a path.
3. Prefer Docker for an always-on server and the desktop app for a single computer. Use the
   standalone installer only when I ask for it.
4. Start Project P and give me the Web URL. Pair the CLI with `project-p login --local`
   for a standalone server, or with the Pairing Code from the logs for Docker.
5. Create one library with `project-p library init` (prefix with `docker exec project-p` in
   Docker), run the initial scan, and verify `project-p api health.get --json` and
   `project-p api library.list --json`.
6. Report the Web URL, data directory, library id, and the next step for adding manga.
7. Ask before deleting, moving, or reorganizing files. Never print a pairing code or access
   token in full.
```

Agent rules:

- Prefer `--json` and check the exit status. The CLI writes workflow JSON for completed
  commands and the raw Contract body for `project-p api`.
- Use `project-p api` for operations that do not have a named command.
- Treat Pairing Codes, access tokens, `secrets.env`, and Source Extension sessions as
  credentials. Do not paste them into issue reports or logs.
- Ask before `manga.delete`, `library.delete`, `download.storage.orphans.delete`,
  `backup.delete`, or any restore operation.
- Back up the data directory before upgrades. Database migrations may not be reversible.
- Do not change a Library Path between `managed` and `referenced` without confirming that
  Project P may write there.

## CLI

`project-p` is both the server and the administration client. Start with `project-p --help` or
`project-p <command> --help`. For a remote Control Plane, pass
`--api-url https://your-host/api` (or set `PROJECT_P_API_URL`) before the command.

| Task                         | Command                                                                                                |
| ---------------------------- | ------------------------------------------------------------------------------------------------------ |
| Serve the Web client and API | `project-p serve --open`                                                                               |
| Pair this CLI                | `project-p login --local`, or `project-p login <pairing-code>`                                         |
| Create a library and scan    | `project-p library init <path> --name <name>`                                                          |
| Rescan a library             | `project-p library scan <library-id>`                                                                  |
| Search a source              | `project-p extension search <key> --query "<text>"`                                                    |
| Import a source candidate    | `project-p manga import <library-id> --library-path-id <path-id> --extension <key> --external-id <id>` |
| Download chapters            | `project-p download <manga-id> --chapter <chapter-id>`                                                 |
| Watch a job                  | `project-p job watch <job-id>`                                                                         |
| Print the OpenAPI Contract   | `project-p openapi`                                                                                    |

### Full CRUD through the API

The named commands are workflow shortcuts. Every Control Plane operation is available through
`project-p api`, including create, read, update, and delete for libraries, manga, collections,
subscriptions, reading state, downloads, backups, and settings.

```sh
# List libraries or read one manga
project-p api library.list --json
project-p api manga.get --param mangaId=<manga-id> --json

# Create a library with its first Library Path
project-p api library.create --json --data '{
  "name": "Manga",
  "libraryPath": {
    "path": "/data/library",
    "ownershipMode": "managed",
    "isDefaultOrganizePath": true
  }
}'

# Update user-owned manga metadata
project-p api manga.metadata.update --param mangaId=<manga-id> --json --data '{
  "title": "Example",
  "status": "ongoing",
  "contentRating": "safe",
  "tags": []
}'

# Create a collection, then add a manga
project-p api collection.create --json --data '{"name":"Favorites"}'
project-p api collection.manga.add --param collectionId=<collection-id> --param mangaId=<manga-id>
```

Use `--paginate` for cursor-paginated list operations, `--input -` for a request body from
stdin, and `--include` when response headers matter. If an operation ID is unknown, use the raw
method and path form:

```sh
project-p api get /library
project-p api patch /library/<library-id> --data '{"name":"Renamed"}'
```

Discover operation IDs with `project-p openapi`; the generated Contract is the source of truth.

### Sources, downloads, and jobs

Project P includes built-in Source Extensions. The list is visible in **Settings → 扩展** or
through `project-p api extension.list`. Use `project-p api extension.get` with
`--param extensionKey=<key>` for one extension's configuration and session state.

```sh
project-p extension search ehentai --query "example" --json
project-p api library.path.list --param libraryId=<library-id> --json
project-p manga import <library-id> \
  --library-path-id <library-path-id> \
  --extension ehentai \
  --external-id <external-id> --json
project-p download <manga-id> --chapter <chapter-id> --json
```

`manga import` waits for catalog import by default and prints the imported Manga. `download`
waits for the Download Batch by default. Add `--no-wait` when an agent will poll with
`project-p job watch` or `project-p api batch.get`.

## Data, backup, and upgrade

- Standalone data defaults to `~/.project-p`; Docker data lives in the `project-p-data` volume.
- Manga files in separate Library Paths are not part of the data directory. Back them up
  separately.
- **Settings → 备份与恢复** creates portable ZIP backups and previews merge or overwrite
  restores. Automatic backups are opt-in.
- Stop the server or container before copying the data directory or replacing a build.
- Database migrations may not be reversible. Keep the pre-upgrade backup until the new version
  has been verified.

## Verify downloads

Every release includes one `SHA256SUMS` file. Compare the hash of the file you downloaded:

```sh
sha256sum project-p-<version>-linux-x64.tar.gz      # Linux
shasum -a 256 project-p-<version>-macos-arm64.tar.gz # macOS
```

The matching line in `SHA256SUMS` must have the same hash. The installers perform this check
automatically.

## Troubleshooting

| Symptom                         | Check                                                                                                             |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Docker asks for a Pairing Code  | Read `docker logs -f project-p`, or run `docker exec project-p project-p pair --public-url http://localhost:8800` |
| Docker cannot see manga         | Mount the host folder and use the container path (`/manga`), not the host path                                    |
| Permission denied on a mount    | The container runs as UID `10001`; fix the mount ownership or use a managed path under `/data`                    |
| Port 8800 is in use             | Change the published port or `PROJECT_P_BIND`                                                                     |
| macOS or Windows blocks the app | Verify `SHA256SUMS`, then use the system approval flow                                                            |
| A scan finds nothing            | Confirm the Library Path contains CBZ/CBR/ZIP files and run `project-p library scan <library-id>`                 |

Logs are under `<data-dir>/logs`; Docker logs are available with `docker logs project-p`.

## Development

Project P is under active development. This repository publishes builds and collects bug
reports; the development source is private. Use the issue tracker for reproducible bugs and
include the Project P version, platform, and relevant logs with secrets and pairing codes
removed.
