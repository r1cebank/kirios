# 17 · Authoritative references and verification notes

Research baseline: **2026-10-12**. Links are starting points, not a substitute for testing against the lockfile commit. Option names and hardware support can change, especially for Gamescope, Quickshell and experimental VM graphics.

## NixOS / declarative system

- NixOS 26.05 stable manual: https://nixos.org/manual/nixos/stable/
- NixOS 26.05 release notes (systemd-based initrd default important for LUKS): https://github.com/NixOS/nixpkgs/blob/master/nixos/doc/manual/release-notes/rl-2605.section.md
- Nix Flakes and secret exposure caveat: https://wiki.nixos.org/wiki/Flakes/en
- Home Manager flakes/NixOS module: https://nix-community.github.io/home-manager/nix-flakes.html
- Home Manager Quickshell options: https://nix-community.github.io/home-manager/options/home-manager/programs/quickshell.html

## ISO/install and storage

- NixOS installation manual: https://nixos.org/manual/nixos/stable/
- Disko repository: https://github.com/nix-community/disko
- Disko quickstart (destructive formatting warning): https://github.com/nix-community/disko/blob/master/docs/quickstart.md
- Disko LUKS/Btrfs example, not production-ready unchanged: https://github.com/nix-community/disko/blob/master/example/luks-btrfs-subvolumes.nix
- nixos-anywhere quickstart and VM test (optional reference rather than mandatory installer backend): https://nix-community.github.io/nixos-anywhere/quickstart.html

## Desktop/session

- Hyprland on NixOS (UWSM plus Home Manager systemd conflict): https://wiki.nixos.org/wiki/Hyprland
- Hyprland UWSM guide: https://wiki.hypr.land/useful-utilities/uwsm/
- Quickshell: https://quickshell.org/

## Gaming and performance

- NixOS Steam including Gamescope session and GameMode: https://wiki.nixos.org/wiki/Steam
- GameMode overview: https://github.com/FeralInteractive/gamemode
- Gamescope: https://github.com/ValveSoftware/gamescope
- Proton: https://github.com/ValveSoftware/Proton
- Jovian-NixOS, SteamOS-like features but AMD/Deck focus: https://github.com/Jovian-Experiments/Jovian-NixOS
- Power profiles daemon: https://gitlab.freedesktop.org/upower/power-profiles-daemon
- MangoHud: https://github.com/flightlessmango/MangoHud

## Development/security

- nix-ld (traditional downloaded binaries): https://wiki.nixos.org/wiki/Nix-ld
- VS Code MCP server setup (confirm exact schema version): https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- SOPS-nix: https://github.com/Mic92/sops-nix
- Distrobox: https://distrobox.it/
- systemd `homectl` real tool (name conflict): https://www.freedesktop.org/software/systemd/man/latest/homectl.html

## Verification checklist for each source

- Source is still maintained, version supports NixOS 26.05 and target architecture.
- NixOS option exists in pinned Nixpkgs revision: check `nixos-option`, `nix eval` or flake evaluation.
- Upstream supports GPU/driver/session as configured, not merely package availability.
- No security guidance assumes `nix/store` protects plaintext secrets.
- Privileged disk examples have been adapted to require device identity checks and explicit confirmation.

## Notable factual constraints

- `homectl` is systemd-homed's tool: use `kirictl`.
- UWSM's Hyprland integration must not be duplicated by the Home Manager Hyprland systemd service.
- Disko formatting is destructive; built-in templates are examples, not safe auto-selectors.
- Home Manager can be embedded as a NixOS module so a system rebuild activates managed home configurations.
- ARM64 NixOS support does **not** imply x86 Steam/Proton support, nor does it imply graphics acceleration for Hyprland under every macOS VM.
