# 19 · KiriOS identity and naming contract

**Status: approved.** Project name is **KiriOS**. This is the canonical brand for the NixOS-based distribution, not a placeholder. Its product identity is independent of any specific kernel, compositor or gaming package.

## Public labels

| Product area | Canonical label | Implementation description |
|---|---|---|
| Operating system | **KiriOS** | NixOS-based, declarative Linux distribution |
| Installer | **Kiri Setup** | Branded live-ISO installer and its TUI/GUI |
| Default desktop | **Kiri Desktop** | Hyprland session with custom shell |
| Shell and widgets | **Kiri Shell** | Quickshell/QML surfaces, panels, overlays |
| First boot | **Kiri Welcome** | Account-independent first-run onboarding |
| Settings | **Kiri Settings** | Graphical configuration and system controls |
| Home synchronization | **Kiri Sync** | Git-backed profile and secrets integration |
| Performance view | **Kiri Performance Center** | Quickshell performance and power UI |
| Management CLI | **`kirictl`** | Sole supported KiriOS-specific control CLI |

The performance service/backend remains a separate privileged or narrowly authorized process, **not** an alias for `kirictl` or the Quickshell UI.

## Internal identifiers

- Human-facing OS label: `KiriOS` (capital K, O, S). Never display `Project OS`, `MyOS`, `NixOS Workstation`, or `project-os` as the product name. References to underlying NixOS remain technically correct.
- Suggested Git repository: `kirios`; documentation handoff archive: `kirios-agent-docs.zip`.
- Machine-readable namespace and default paths: lowercase `kirios`, e.g. `$XDG_CONFIG_HOME/kirios/preferences.json`, `/etc/xdg/kirios/defaults.json`, `/var/lib/kirios/install-journal.json`.
- CLI: `kirictl` throughout commands, app launchers, automated tests, example JSON, generated docs and package outputs. Do **not** introduce `deskctl`; do **not** commandeer systemd-homed's existing `homectl`.
- Flake output examples: `packages.<system>.kirictl` and `nixosConfigurations.{xps,thinkstation,arm-vm,vm-test}`; host IDs are not brand names.
- Sample ISO artifact names: `kirios-installer-x86_64.iso` and `kirios-installer-aarch64.iso` (real output name can additionally include architecture and semantic release version).
- D-Bus draft: `org.kirios.Performance1` at `/org/kirios/Performance1`, interface `org.kirios.Performance1`. Version interfaces before shipping.
- Package prefixes and executable names should use `kirios-` where appropriate; avoid overriding binaries provided by NixOS/systemd. Package names still need to be verified for collisions before publishing.
- Service units, user systemd units, desktop files and settings schemas should use the `kirios` namespace consistently.

## Agent acceptance checks

1. Search implementation files for former placeholder identity and CLI aliases. Only the approved `KiriOS` / `kirios` / `kirictl` family may appear in new product-facing identifiers; normal upstream NixOS/systemd names are unaffected.
2. `rg -n 'kirictl|KiriOS|kirios' .` identifies product, CLI and namespace across installer, desktop, IPC and sync components.
3. The ISO splash, installer title, greeter selection, Welcome UI, Settings title, shell About panel, README and release descriptions all use **KiriOS**.
4. No personal identity, dotfiles or secrets are embedded in the publicly shareable KiriOS image.
5. Brand/name-only changes must not silently alter disk behavior, privilege boundaries, schema versions or target hardware constraints.

## Not decided by the name

A logo, palette, iconography, typography, and promotional tagline are **not yet approved**. Initial placeholder visual styling should remain replaceable. Do not imply a domain, trademark, GitHub organization or package repository is registered until confirmed.
