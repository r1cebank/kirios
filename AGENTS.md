# KiriOS | AGENTS.md: mandatory instructions for implementation agents

**Product name is KiriOS.** Use lowercase `kirios` for package prefixes, local paths and service identifiers, and `kirictl` for the single public management CLI. Apply `docs/19-brand-and-naming.md` when naming anything new. Do not introduce old placeholders or a second CLI.


## Priorities

1. **Do not destroy user data.** Every disk operation starts in a simulated/test VM. Never partition a host disk without physical disk identity checks, an explicit typed confirmation, and a clear data-loss warning.
2. **Build working slices.** Do not scaffold every module and leave nothing bootable. Prioritize boot -> network -> install -> reboot -> desktop -> sync -> controls -> gaming.
3. **Prefer upstream NixOS modules.** Reuse official NixOS + Home Manager options before building wrappers. Check option names against pinned Nixpkgs/Home Manager revisions.
4. **No unverified commands as facts.** Examples in this pack are contracts or pseudocode when marked. Validate against pinned sources with `nix eval`, `nix flake check` and end-to-end tests.
5. **Keep secrets out of Git, ISO, build logs and `/nix/store`.** No passwords, access tokens, private keys or Wi-Fi secrets in Nix strings evaluated into world-readable store paths.
6. **Preserve system state model.** Declarative files and user-writable runtime preferences must have distinct owners.
7. **Use stable kernel baseline.** Experimental schedulers, v3/v4 builds, kernel patches and additional power managers must be feature flagged, tested, and reversible.
8. **Support both architectures in evaluation.** Gate x86 gaming, proprietary drivers, vendor tools, and kernel variants with `system == "x86_64-linux"` or a host-specific condition.
9. **Propose small changes and report evidence.** Include implemented files, tests performed, observed failures, risks, and next tasks.

The personal profile integration design is specified in `docs/18-personal-profile-integration.md`; do not assume a raw Git clone is automatically imported by the system flake.

## Initial deliverable: end-to-end x86_64 VM installer

- Flake and lockfile, native NixOS config, test ISO, minimal TUI, Disko integration, noninteractive installation stage guarded by interactive confirmation, NixOS first boot, basic Hyprland + UWSM, Quickshell panel, and a small smoke test.
- Do not declare a milestone complete because only Nix evaluation passed. Verify boot and installation/reboot behavior in QEMU/UEFI.

## Engineering conventions

- Maintain `flake.nix` as the dependency entrypoint. Pin inputs; align `home-manager` release with NixOS release. Update lockfile intentionally.
- Create `nixosModules` for reusable capabilities; host modules only express hardware identity and enabled capabilities.
- Give every module an `enable` option; include assertions for mutual exclusions and architecture constraints where appropriate.
- Use test-driven contracts for CLI stdout/stderr, exit codes, JSON schema and D-Bus APIs.
- Any root-level action must use an explicit privilege boundary with a narrow allowlist; never expose arbitrary shell execution via the Quickshell widget.
- For scripts, use strict error handling, shellcheck/formatters, and exit-status checks. Prefer language runtimes already present in the ISO.
- Each significant design deviation is a markdown ADR under `docs/adr/` in the implementation repository.
- Avoid global world-writable configuration files or user-triggered rewrites of `/etc/nixos` without validation, diff preview and privilege authorization.

## Communication rhythm

At each milestone report: implemented, tested (with exact commands/environment), known blockers, unresolved host hardware facts, security implications, updated roadmap. Ask for hardware-specific facts only when the stage genuinely needs them.

## Terminology

- **MVP:** bootable x86_64 VM installation with working desktop, not the fully finished distribution.
- **Core profile:** safe defaults available everywhere.
- **Developer profile:** Nix-managed stable tools + compatibility support.
- **Gaming profile:** x86_64 Steam/Proton and optional console session.
- **Runtime preferences:** mutable per-user settings in XDG configuration/data paths.
- **Sync:** pull/push/restore with explicit conflict resolution; not an unbounded bidirectional merger.
