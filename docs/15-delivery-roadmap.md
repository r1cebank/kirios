# 15 · Milestones, tasks, gates and dependencies

Use these as a **sequential implementation backlog**. Do not begin a later milestone while the prior acceptance gate is failing. On every milestone, save evidence in `tests/reports/` and update the README.

## M0 · Foundation and evaluation

**Goal:** a working flake/configuration skeleton, not a fake distribution.

- [ ] M0-01 Create flake outputs (`xps`, `thinkstation`, `arm-vm`, `vm-test`) and lockfile for NixOS 26.05/Home Manager 26.05.
- [ ] M0-02 Add a minimal core module, generic VM hardware, users/hostname template, UEFI boot and network management.
- [ ] M0-03 Create architecture gates for graphics/gaming/NVIDIA/certain kernels.
- [ ] M0-04 Add flake evaluation and formatting checks in CI.
- [ ] M0-05 Document host unknowns and ADRs; ensure no credentials/source paths in store.

**Gate:** `nix flake check` and host evaluation for both architectures; minimal x86_64 VM boots.

## M1 · Bootable network installer (first usable slice)

- [ ] M1-01 Build x86_64 custom install ISO from stock NixOS installer module.
- [ ] M1-02 Implement TUI wizard and no-write connectivity/hardware/disk preflight.
- [ ] M1-03 Implement schema-validated install plan + preview + typed target erase confirmation.
- [ ] M1-04 Implement one simple Disko GPT + ESP + Btrfs layout; add optional LUKS after unencrypted path passes.
- [ ] M1-05 Install pinned target closure and validated generated hardware config; reboot into TTY/greeter.
- [ ] M1-06 Test failure paths and emit redacted install journal.
- [ ] M1-07 Build ARM64 ISO and VM console installer after x86 flow works; do not rely on GPU.

**Gate:** disposable UEFI x86_64 VM install->reboot, no credential leaks, wrong-disk guard tests; ARM64 ISO boot if test runner available.

## M2 · Functional Hyprland desktop

- [ ] M2-01 UWSM Hyprland + Xwayland + portal + sound + clipboard + screenshot + lock/idle.
- [ ] M2-02 Basic greeter and fallback session, terminal/file manager/browser.
- [ ] M2-03 Minimal Quickshell top bar and launcher; user systemd service.
- [ ] M2-04 Shared theme tokens, mutable XDG preference store and live reload.
- [ ] M2-05 Basic settings/welcome app scaffold with skip button.

**Gate:** fresh boot from ISO into usable session; preferences survive restart/rebuild; CLI/TTY recovery.

## M3 · Home Sync and restore

- [ ] M3-01 Integrated Home Manager declarative package/dotfile defaults.
- [ ] M3-02 Git clone/fetch/status, dry-run and conflict-safe backup flow.
- [ ] M3-03 Per-host overlays and public/private user profile separation.
- [ ] M3-04 Optional age/SOPS unlock with no plaintext store leakage.
- [ ] M3-05 Welcome integration; robust skip/offline handling.

**Gate:** two clean installations from same pinned commit match declared state; conflict tests preserve originals.

## M4 · `kirictl` CLI

- [ ] M4-01 Rust CLI skeleton, JSON envelope, error taxonomy and config schema.
- [ ] M4-02 `status`, `doctor`, `hardware probe`, `profile`, `theme`, `widgets` commands.
- [ ] M4-03 `apply --dry-run`, diff, privileged activation, rollback/generations.
- [ ] M4-04 `sync` commands and Welcome app backend integration.
- [ ] M4-05 Unit/integration tests, shell completion, man/help pages.

**Gate:** CLI modifies a profile via validated manifest, shows diff, applies, reports current generation, rolls back in VM.

## M5 · Performance engine and widget

- [ ] M5-01 Gather host capabilities and readings (CPU/RAM, battery, optional GPU/sensors).
- [ ] M5-02 Build policy state machine with precedence, notifications and timeout/revert behavior.
- [ ] M5-03 Bind safe power backend; avoid conflicting control services.
- [ ] M5-04 Expose read-only/status + narrow mode request API with authorization.
- [ ] M5-05 Quickshell compact/flyout/full Performance Center UI.
- [ ] M5-06 Benchmark polling overhead, unsupported ARM behavior, game exit/crash restore.

**Gate:** profile switches tested on laptop/workstation with UI/backend agreement, no privileged shell command injection.

## M6 · Gaming and compatibility

- [ ] M6-01 x86_64 Steam/Proton and Vulkan/32-bit graphics modules.
- [ ] M6-02 GameMode/MangoHud launch and exit validation.
- [ ] M6-03 Optional Gamescope console session with NVIDIA-specific tests and greeter fallback.
- [ ] M6-04 Node/NVM/VS Code/MCP smoke suite; nix-ld and explicit runtime wrappers.
- [ ] M6-05 Podman/Distrobox fallback; traditional SDK testing.

**Gate:** known compatible Proton game runs on physical supported GPU; GUI VS Code launches tested MCP Node subprocess.

## M7 · Physical hardware and performance polish

- [ ] M7-01 XPS host probing: GPU/Wi-Fi/battery/suspend/thermal, tune only measured regressions.
- [ ] M7-02 ThinkStation GPU identity, NVIDIA driver + 32-bit Vulkan, CUDA optional, Gamescope validation.
- [ ] M7-03 Stock-kernel performance baseline then controlled variants; record reproducible datasets.
- [ ] M7-04 ARM64 VM fallback graphics and boot/install evidence.
- [ ] M7-05 Accessibility, robustness, long-run tests, local backup drills.

**Gate:** hardware matrix signed off; no blocking regressions versus stock baseline.

## M8 · Publishing and long-term maintenance

- [ ] M8-01 Signed release metadata, ISO checksum/signature and versioning.
- [ ] M8-02 x86_64 and aarch64 CI release builds with reproducibility evidence.
- [ ] M8-03 Optional x86 builder/cache on ThinkStation; ARM builder on macOS Linux VM.
- [ ] M8-04 Updates UI, changelog, generation protection and safe cleanup policy.
- [ ] M8-05 End-user recovery docs, non-goals and unsupported-game/hardware disclosures.

**Gate:** a fresh user can verify ISO, install, restore optional home, run diagnostics and recover previous generation.

## Critical path

`M0 -> M1 -> M2 -> M3 -> M4 -> M5 -> M6 -> M7 -> M8`.

Small user-facing additions can happen in parallel **after M2**; do not build complex QML dashboards before installer, desktop session and state model are functional. A bounded optional Phase M1a for LUKS can complete after first unencrypted install test.

## Definition of complete per ticket

Code reviewed, tests green in actual target environment, docs updated, rollback/error path explained, and hardware caveats captured. No “works on my terminal” acceptance for VS Code/GUI/MCP integration.
