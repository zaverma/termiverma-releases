# Termiverma 0.6.4 — Factory workspace and Linux desktop improvements

## A clearer Factory workspace

- **Selected devices and the playbook/snippet now use compact summary cards.**
  Edit the selection or open a read-only resource preview without expanding the
  whole preparation page. The layout adapts to narrower windows.
- **Execution has its own workspace and separate Live log.** Device status,
  the current stage, cancellation and output are easier to follow. Overall
  progress reflects completed devices; running devices show activity without
  inventing a completion percentage.
- **Snippet jobs can explicitly continue after CLI errors.** The new
  **Continue after CLI errors** option is off by default and is captured in the
  run review. With it enabled, remaining commands are sent after a verified
  prompt, while any observed CLI error still produces a failed result.
  Connection/access failures, paging, credential requests, timeouts and
  cancellation still stop execution. Ansible behavior is unchanged.
- **An empty Factory jobs section is hidden when Factory is unavailable.**
  Existing admitted jobs remain available to inspect or cancel after license
  expiry; the change does not remove their results or grant new-job access.

## Linux desktop improvements

- **One window header with Minimize, Maximize/Restore and Close.** The extra
  system titlebar is removed for the main window and additional Workspaces.
  Close uses the normal window/session cleanup path; resizing and dragging
  retain the native Tauri/GTK handling.
- **Application-menu shortcuts are now an explicit choice.** A release AppImage
  offers **Create shortcut / Not now** once, after unlocking. The choice is
  remembered and other dialogs finish before the offer appears.
  **Settings → About → Application shortcut** can create, restore, update or
  remove the app-managed shortcut. Update it there after moving the AppImage.
  Existing package/custom launchers are preserved; no launcher is created
  automatically at startup.
- **Invalid `XCURSOR_SIZE` overrides are normalized before GTK starts.** Zero,
  negative, malformed and unusably large explicit values fall back to 24.
  Valid or unset sizes, cursor themes and the selected display backend remain
  unchanged.

## Documentation

- Added a local Ansible preparation guide for macOS and Linux, including the
  required Python/libssh environment, collections and troubleshooting.
- Current work now has one stable task register and separate Free/Pro/Team
  roadmaps. Earlier planning snapshots and release notes live in the archive.

## Verification and release status

Automated frontend, Rust and build checks cover these changes. Native Linux
GNOME/KDE menu behavior, X11/Wayland window interaction and the affected
AppImage startup still require platform retesting. Factory's physical Cisco
workflow and visual desktop acceptance remain separate from synthetic fixtures.

Source version **0.6.4**, macOS native build **26**, tag **v0.6.4**. The root
`latest.json` template contains the version, notes and UTC publication date;
platform URLs and signatures remain blank until fresh **0.6.4** artifacts are
built and signed. The source commit and tag do not publish installers or a
public binary release.
