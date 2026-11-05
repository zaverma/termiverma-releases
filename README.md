# Termiverma

**SSH, Telnet and local terminals for network engineers on macOS and Linux.**

Termiverma brings terminal sessions, device inventory, saved accounts and command
snippets into one desktop workspace.

[Download the latest release](https://github.com/zaverma/termiverma-releases/releases/latest)
· [All releases](https://github.com/zaverma/termiverma-releases/releases)
· [Website](https://zaverma.com)

This repository distributes application binaries, update metadata and release
notes. Termiverma is proprietary commercial software by **zaverma**; its source
code is not distributed here.

## Features

### Connections and terminal workspace

- Connect over SSH or Telnet from Quick Connect, a saved host or connection
  history. Enter an address directly or explicitly select an inventory suggestion
  by a partial device name or address. A typed address is used as entered, with
  an IP's CIDR suffix removed before connecting; the app does not substitute
  another inventory address automatically.
- SSH negotiates modern algorithms first and includes a bounded compatibility
  policy for supported older network equipment, including legacy Cisco devices.
  No per-host algorithm settings are required. Host keys are checked during the
  handshake: first use asks for trust, recognized keys proceed, and changed keys
  block authentication until the old and new fingerprints are reviewed and the
  replacement is explicitly accepted.
- SSH supports password, keyboard-interactive and supported local identity-file
  or SSH-agent authentication. Saved password accounts are managed in Vault;
  storing private keys inside Accounts is not included.
- Use separate **Connections** and **Local** tab groups. Local opens an interactive
  shell in your home directory. Switching groups retains running sessions, tab
  order and the selected tab. Drag tabs to reorder them.
- Open multiple application windows with independent terminal workspaces. Windows
  share inventory, history, snippets and Vault/security state. Closing a window
  closes its sessions while sessions in other windows continue.
- Choose among six remote-tab title formats using address, username and inventory
  hostname, or give any tab a custom name. The actual endpoint remains available
  in its tooltip.
- Search the active terminal's available output and scrollback. Use independent
  terminal color palettes and optional IPv4, IPv6, MAC, URL and hostname
  highlighting.
- When a remote session ends, its output remains behind a blurred reconnect
  overlay. Reconnect uses the same destination and saved-account reference when
  available; **Close tab** is available in the same overlay.

### Hosts and NetBox

- Maintain local devices with roles, sites, site groups, regions, platforms,
  status and tags. Search, filter, group and sort the catalogue; choose visible
  table columns and inspect a device in a resizable side panel.
- Sync one read-only NetBox source using Primary IP, with configurable device
  role/status filters and address preference. Available Site Group and Region
  hierarchy is included in the inventory.
- Search and connect from the last complete offline NetBox cache. Local account
  and connection overrides survive successful inventory refreshes. Sync is
  started explicitly; it does not write inventory changes back to NetBox.
- Use one default SSH account across devices, choose an account per device, or
  sign in manually. Recent connections show the inventory name and exact address
  when the name is unambiguous.

### Accounts and snippets

- Store named username/password accounts and integration tokens in a locally
  encrypted Vault. The same master password unlocks Vault and the optional
  application lock. Turning off the UI lock keeps Vault encrypted at rest;
  application and Vault inactivity timeouts are configurable.
- Create local snippets and nested folders, or sync multiple read-only Git
  snippet repositories while preserving their directory structure. Public and
  private HTTPS repositories are supported using installed Git credentials or
  an access token stored in Vault. Repositories can be removed from the app
  without deleting the remote repository.
- Keep a cached Git library available offline. Run commands explicitly from the
  library or a resizable panel beside a connected remote or Local terminal.
  Folder expansion and panel scroll position are remembered.
- **Protected** snippets require confirmation of the target and final command
  before each run. Synchronizing, selecting a snippet or opening the application
  does not execute commands. Git content remains read-only in Termiverma.

### Appearance and updates

- English and Ukrainian interfaces; Light, Dark and System appearance.
- Signed in-app updates for supported macOS application bundles and Linux
  AppImages, with download progress, signature verification and a separate
  restart choice. Settings also provides a manual update check.
- Linux AppImages register an application launcher and icons at the user level,
  allowing Termiverma to be pinned to the desktop environment's dock/favorites.

## Installation

Download assets only from the [release page](https://github.com/zaverma/termiverma-releases/releases/latest)
and choose the package for your operating system and CPU architecture. Packages
are listed on each release; a format or architecture mentioned here is not a
promise that it is available in every release.

### macOS — Apple Silicon

1. Download the `.dmg`, open it and drag **Termiverma.app** into **Applications**.
2. Eject the disk image and launch Termiverma from Applications.

The current macOS distribution targets Apple Silicon. It uses ad-hoc application
signing and is not Apple-notarized; macOS may restrict the first launch. If you
choose to allow it, use the per-application approval in **System Settings →
Privacy & Security** after the blocked launch. Availability of that approval can
depend on the Mac's security policy. The updater's signature is separate from
Apple Developer ID signing and notarization. See
[Apple's guidance on opening downloaded apps](https://support.apple.com/en-us/102445).

### Linux — AppImage

Download the AppImage for your architecture, place it in a permanent location,
make it executable and run it. For example, in the download directory:

```sh
chmod +x ./Termiverma_*.AppImage
```

Launch the downloaded AppImage from your file manager. After its first run,
Termiverma adds a user-level desktop launcher and application icons; pin it using
your desktop environment's normal dock/favorites action. Moving the AppImage and
running it from the new location updates the app-managed launcher.

In-app Linux installation of updates applies to **AppImage**. If a release also
provides `.deb` or `.rpm` packages, install and update those using the relevant
package tools; they do not use the AppImage update path. Linux builds are
prepared separately, and compatibility can vary with the distribution and
desktop environment.

Git synchronization requires an installed `git` client. Telnet requires an
installed `telnet` client available to the application.

## Keyboard shortcuts

| Action | macOS | Linux |
| --- | --- | --- |
| New tab in the selected Connections / Local group | ⌘T | Ctrl+T |
| Close the active tab | ⌘W | Ctrl+W |
| New application window | ⌘Shift+N | Ctrl+Shift+N |
| Search the active terminal output | ⌘F | Ctrl+F |
| Next / previous search match, from the search input | Enter / Shift+Enter | Enter / Shift+Enter |
| Close terminal search | Escape | Escape |
| Rename a focused tab | F2 | F2 |
| Copy selected terminal output | ⌘C | Ctrl+Shift+C, or Ctrl+C with a selection |

On Linux, Ctrl+C without a selection retains its normal terminal interrupt
behavior. Tab shortcuts act on the current window and group; the permanent
Quick Connect home is not closed by Close tab. A tab can also be renamed by
double-clicking it or using its context menu. macOS additionally offers **File →
New Window** and **New Window** in the Dock menu.

## Updates and download files

Use **Settings → Updates** to check for a newer version. Installation requires an
eligible packaged build and a published update for its platform. An update does
not restart the application without your choice; restarting ends open sessions
across all Termiverma windows.

Release assets may include `.sig`, `latest.json` and `SHA256SUMS` alongside the
installers. Signature and manifest files support the in-app updater; they are
not applications to open. `SHA256SUMS` lets you compare downloaded files with the
published checksums. Signature verification remains mandatory before an in-app
update is installed.

## Scope and ownership

Version 0.4 does not include serial/USB console access, SCP/SFTP transfers,
terminal splits, bulk snippet jobs, data import/export or KeePassXC integration.
Git synchronization is read-only; publishing snippets to Git from the app is
outside its scope.

Product information and contact: [zaverma.com](https://zaverma.com).

Copyright © 2026 zaverma. All rights reserved. Termiverma is proprietary software;
public access to release files does not grant a license to its source code.
Third-party components retain their respective licenses and notices.
