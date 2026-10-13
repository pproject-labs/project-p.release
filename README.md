# Project P

Project P is a personal media library, currently focused on manga.

This repository hosts desktop installers, standalone executable downloads,
checksums, release notes, and issue reports. Development source is private.

## Downloads

Download previews from [Releases](https://github.com/pproject-labs/project-p.release/releases).
Current previews use the nightly channel. Stable releases will be listed separately.

Desktop packaging targets macOS Apple Silicon, Windows x64, and Linux x64.
Preview packages are unsigned and are not notarized. They do not currently
install updates automatically.

## Install and upgrade

Verify your download against the matching SHA256SUMS manifest.
Quit Project P before replacing an installed version, and back up your library
data before an upgrade. An upgrade can migrate the database; restoring an older
application may also require restoring its matching data backup.

On macOS, install the app from the DMG into Applications. On Windows, run the
NSIS installer. Installed library data is kept separately from the application.

## Report a problem

Open an issue with your application version, operating system, and steps to
reproduce the problem. Remove personal paths, credentials, and media information
from logs before sharing them.
