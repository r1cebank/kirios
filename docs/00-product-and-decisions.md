# 00 · KiriOS product brief and architecture decisions

## User story

“I boot a custom ISO, connect to the internet, select a disk and profile, install quickly, reboot into a polished Hyprland/Quickshell desktop, then sign in to Git and restore my personal environment. I can manage software, themes, sync and adaptive performance without memorizing Nix syntax. Steam + Proton should work like a gaming PC and a console-style session should be available when supported.”

## Must-have goals

- A reproducible NixOS installation across Dell XPS (i7-12700H, 64 GB), ThinkStation (20-core Xeon W5, 256 GB, RTX 6000 exact model TBD) and ARM64 Linux VM under macOS.
- Personalized but shareable system: branded ISO and useful defaults with **no private profile baked in**.
- Network installer with install source pinned to a release/commit; clear progress, resume-safe stages, offline diagnostics and an emergency shell.
- First boot already functional (login, compositor, network, terminal, file manager, settings); sync is optional.
- Home Manager-managed user environment with distinct mutable Quickshell settings.
- `kirictl` lifecycle commands and optional desktop UI using same backend.
- Efficiency/performance profiles based on measured telemetry and safe hardware abstractions.
- Steam/Proton desktop mode, optional Gamescope gaming session, GameMode and MangoHud for x86_64.
- Ordinary developer workflows: Node/npm/nvm, VS Code launched from a desktop menu, MCP stdio subprocesses, Python, Rust/C++, containers and downloaded binaries.

## Non-goals for v1

- Reproducing all SteamOS internals, guaranteed Proton compatibility, console suspend/resume parity, anti-cheat bypasses.
- GPU overclocking, undervolting, firmware flashing, fan curve writes, experimental scheduler/kernels by default.
- Automatic AUR compatibility on NixOS; forcing Nix into an Arch-style FHS globally.
- Full disk dual boot or preservation of preexisting partitions in the first guided destructive installer.
- Secret recovery without authentication/decryption keys; syncing private application databases blindly.
- Perfect GPU-accelerated Hyprland in macOS ARM virtual machines.

## ADR-000: Project identity is KiriOS

The distribution, installer and release artifacts use **KiriOS** as the public brand. Use lowercase `kirios` as internal namespace. The public management CLI is **`kirictl`**; see `docs/19-brand-and-naming.md`. This decision supersedes all former placeholder names.

## ADR-001: Choose NixOS, not Arch + Nix

The primary user priority is declarative *whole-system* reproducibility and rollback. NixOS is the source of truth for packages, services and boot; a compatibility tier avoids making everyday development dependent on writing Nix derivations.

## ADR-002: Standard kernel before performance variants

Begin with stock supported kernels and NVIDIA-compatible drivers. Benchmark alternatives under comparable conditions. Include a fallback generation and stable kernel. Never let “fast” imply constant max CPU clocks.

## ADR-003: Integrated Home Manager + mutable preferences

Home Manager is integrated into `nixos-rebuild`; it owns reproducible files and user packages. Quickshell runtime preferences live in `$XDG_CONFIG_HOME/kirios/preferences.json` (owned by user, not a read-only Home Manager symlink). Build initial defaults at login only if absent; allow explicitly importing/exporting preferences. See `07`.

## ADR-004: `kirictl` CLI, not `homectl`

`homectl` is an established systemd-homed tool. `kirictl` is the canonical KiriOS management command. Aliases should not shadow systemd executables.

## ADR-005: Console mode is a session, not the default

Hyprland desktop is the default; Gamescope/Steam session is selectable on validated x86_64 GPUs. Session failure returns to greeter; no autologin to games.

## ADR-006: Three stable hardware identities

Named host definitions: `xps`, `thinkstation`, `arm-vm`. Install target discovery MUST NOT infer destructive disk targets from a model name. GPU details remain configurable, and an unknown host starts with conservative generic drivers.

## Acceptance outcomes

1. Fresh unattended *VM-only* reinstall from same commit matches package generations and expected configs.
2. No surprise changes to existing disk or user config during sync/update.
3. Desktop is usable while offline after initial installation and after first boot without sync.
4. Every performance mode has deterministic restoration after app exit, crash, logout, or service restart.
5. Compiler/AI-agent runtimes are independently testable from GUI-launched apps.
