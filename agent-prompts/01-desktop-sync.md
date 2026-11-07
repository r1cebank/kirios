# Prompt 01 · Build a complete desktop and home restoration

Read `AGENTS.md`, `docs/06-desktop-and-theme.md`, `docs/07-home-sync-and-secrets.md`, `docs/05-hardware-and-platforms.md` and the M2/M3 acceptance gates. Assume M0/M1 VM install has passed.

Implement a production-minded UWSM-managed Hyprland desktop, Xwayland/portal/audio/lock utilities, a greeter/fallback, minimal modular Quickshell top bar/launcher/settings, live XDG preference store, and a welcome wizard with skip/offline handling. Add integrated Home Manager with compatible pinned release, conflicts/backup-preview behavior, personal Git checkout flow and optional encrypted secret restore. No personal Git profile should be embedded in the public ISO. Do not create a read-only Home Manager symlink for a live-editable preferences file.

Test a fresh installed VM login, no-network/skip path, restart of Quickshell, preference persistence after NixOS rebuild, sync conflict preservation, and cross-host Home Manager evaluation. Report exact observed tests and source revisions.
