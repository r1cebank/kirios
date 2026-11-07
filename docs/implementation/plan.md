# KiriOS Implementation Plan

Date: 2026-11-07. Status: ready for Linux-host implementation; no implementation work started.

## Goal and Scope

Build a reproducible NixOS distribution through working, testable slices: boot -> network -> install -> reboot -> desktop -> sync -> controls -> gaming. Preserve the KiriOS brand, `kirios` namespace and sole public management CLI, `kirictl`.

The first implementation delivery is M0 followed by M1: a pinned system and a guarded x86_64 VM installer. This is an **installer milestone**, not the MVP. The MVP is reached after M2, when that same installation path boots a usable desktop with Hyprland/UWSM, a minimal Quickshell panel and `kirictl doctor`. [ADR 0007](../adr/0007-delivery-boundaries-and-early-cli.md) explains the sequencing.

The current task ends with planning. Resume implementation on the Linux environment described in [the handoff](README.md). No physical-machine installation, personal-profile access, release publishing or experimental tuning is part of the first delivery.

## Specification Traceability

| Plan concern | Authoritative section |
|---|---|
| Delivery and safety | [AGENTS: Priorities; Initial deliverable](../../AGENTS.md), [00: Must-have goals; Non-goals for v1](../00-product-and-decisions.md) |
| State ownership and privilege | [01: Configuration ownership table; Boundaries; Security trust boundaries](../01-architecture.md) |
| Flake and modules | [02: Flake outputs required; Flake inputs; Host composition rules](../02-flake-and-repository.md) |
| Installer | [03: Execution phases; Disk path protection](../03-iso-and-installer.md), [04: v1 target disk layout; Recovery routes](../04-disk-and-recovery.md) |
| Architectures and unknown devices | [05: Hardware matrix and host-specific policy](../05-hardware-and-platforms.md) |
| Desktop and mutable preferences | [06: Desktop contract; Live preference ownership; Fallbacks](../06-desktop-and-theme.md) |
| Sync and safe activation | [07: Three kinds of home state; Sync flow](../07-home-sync-and-secrets.md), [18: Selected v1 design](../18-personal-profile-integration.md) |
| Public API and privileged controls | [08: Exit-code contract; JSON envelope](../08-kirictl-cli-and-config.md), [09: Data architecture](../09-performance-center.md), [10: State machine; Polkit and privilege design](../10-performance-engine.md) |
| Later workloads | [11: Scope; Two sessions](../11-gaming-stack.md), [12: Runtime layers; Test suite](../12-developer-experience.md) |
| Trust, evidence and ordering | [13: Trusted sources; Secret handling](../13-updates-and-security.md), [14: Test pyramid](../14-test-plan.md), [15: Milestones](../15-delivery-roadmap.md), [16: Work style](../16-agent-workflow.md) |
| Source verification and naming | [17: Verification checklist for each source](../17-sources-and-verification.md), [19: Internal identifiers](../19-brand-and-naming.md) |

## Architecture to Implement

Use one pinned flake, integrated Home Manager and upstream modules wherever available. Start with the specified NixOS 26.05 and Home Manager release-26.05, with Home Manager following the Nixpkgs input. Select actual commits and verify option names in M0; no commit or lockfile has been fabricated during planning. Disko is the only additional required installer input initially.

Use **Limine** for installed-system UEFI boot, as explicitly selected by the user; see [ADR 0008](../adr/0008-limine-bootloader.md). Verify its upstream integration and both-architecture configuration against the lock in M0. Test current/previous generation boot, installed-disk boot without the ISO and later LUKS unlock before claiming those capabilities. Inspect and document the upstream live ISO's separate boot chain.

Expose reusable capabilities as opt-in NixOS modules with `enable` options and capability assertions. Host modules compose capabilities and hardware identity. All four named hosts must evaluate; physical host outputs remain conservative templates until real hardware data exists, not installations ready for deployment.

Keep these boundaries explicit:

- The TUI collects answers, probes safely and displays a validated plan. It does not perform destructive actions.
- A separate privileged executor validates the plan again, checks source trust, binds consent to the reviewed plan and current device identity, and runs a narrow install sequence. It accepts no arbitrary command strings or user-supplied Nix expressions.
- The flake owns system configuration and integrated Home Manager owns declared home files. XDG runtime preferences belong to the user and are never replaced by managed symlinks.
- The public CLI and desktop use shared typed contracts. Early read-only CLI commands support testing; the broad command surface remains M4.
- Personal Git is data in restricted mode. M3 validates manifests/assets and stages an approved overlay; it never automatically imports arbitrary `home.nix` code.
- Power controls use supported upstream services first. Additional orchestration must justify its privilege and API surface.

Create directories only when the owning slice needs them. Follow [the proposed repository tree](../02-flake-and-repository.md); do not populate empty subsystems in advance.

## Milestones and Gates

All milestones below are **not started**. The detailed first slices are in [the backlog](backlog.md). Never treat evaluation, mock data, an ISO build or a boot screenshot as evidence for a later installation/desktop claim.

| Stage | Deliverable | Required acceptance gate |
|---|---|---|
| M0 Foundation | Locked flake, core modules, conservative host definitions, VM hardware, dev shell and CI | All four hosts evaluate; architecture assertions pass; native x86_64 build boots in QEMU/UEFI with network |
| M1 Installer | Stock-based ISO, safe TUI, reviewed InstallPlan, Disko GPT/ESP/Btrfs layout, runtime account setup, pinned install and redacted journal | Real ISO installs disposable x86_64 disk, ejects ISO, reboots into installed TTY/login, works offline; wrong-disk and failure gates pass |
| M2 Desktop / MVP | Integrated Home Manager defaults, greeter, UWSM/Hyprland, portal/audio/basic apps, minimal Quickshell, persistent preferences, Welcome skip, read-only doctor | Fresh M1 installation reaches usable desktop on a supported virtual GPU; panel and doctor work; offline login and TTY recovery pass; preferences survive rebuild |
| M3 Sync | Restricted personal manifest, conflict preview/backups, per-host overlays, optional encrypted restore | Two clean installations reproduce declared state; conflicts preserve original bytes; hostile paths and wrong decryption keys fail safely |
| M4 Management | Full typed CLI, manifest editing, diff/apply, authorized activation, generations and rollback | CLI changes a validated profile, previews, activates and rolls back in VM; failures and concurrent applies preserve consistent state |
| M5 Performance | Capabilities/telemetry, reversible policy, upstream power integration and shell widget | Mock failure tests pass; supported real hardware demonstrates requested/effective mode agreement and restoration after exit/crash |
| M6 Gaming / developer compatibility | Architecture-gated Steam/Vulkan, GameMode/MangoHud, optional Gamescope; GUI Node/VS Code/MCP and development workflows | Supported physical GPU runs a compatible Proton game; GUI VS Code spawns the tested Node MCP fixture; gaming exit/fallback works |
| M7 Hardware qualification | Exact host drivers, suspend/network/audio/display checks, recovery drills and measured performance | XPS/ThinkStation matrix signed off against stock baseline; ARM guest boot/install evidence and graphics tier recorded |
| M8 Release / maintenance | Signed releases, two-architecture builds, update UX, retention and end-user recovery | Verified ISO can install, optionally restore a home, diagnose and recover a previous generation; published limitations match evidence |

M0 -> M1 -> M2 is the immediate critical path. Continue through M3-M8 only after previous gates pass. M5 real-host power validation requires read-only hardware discovery before M7's full qualification; it does not authorize early physical disk writes. Likewise M6 GPU tests precede M7's complete hardware sign-off.

## Architecture and Feature Schedule

Evaluate x86_64 and aarch64 from M0. Keep x86 gaming, vendor GPU packages and experimental kernels out of ARM outputs. Build and test the x86_64 installer first. Then build the ARM ISO on a native ARM64 Linux runner and validate a UEFI console installation when that runner exists. Missing ARM runtime evidence must remain explicit; it blocks claims of ARM readiness, not the independently evidenced x86 installer gate.

Unencrypted Btrfs is the first disposable-VM path only. Add LUKS as bounded M1a after that flow passes, before recommending laptop installation. Do not offer an unimplemented encryption choice. Validate initrd/unlock behavior from locked sources; test incorrect keys and reboot recovery.

For M2 choose a virtual graphics configuration that can actually exercise Hyprland. An unsupported GPU's fallback proves recovery, not the Hyprland MVP. ARM may remain console-only until its graphics capabilities are demonstrated.

## Working Rules

Before each slice: read relevant source, lock the contract, implement one observable path, run its failure cases, and record actual evidence. Use Rust as the planned CLI/installer implementation language; start with a small TUI and shared validation types. Record a change in language or privilege architecture as an ADR if source/toolchain constraints require it.

Stop on failed safety or acceptance gates. For infrastructure gaps, record the missing runner/tool and the checks not run; do not mark the gate passed. Do not install toolchains or run VM work in the present planning session.

At each milestone record files changed, exact commands/environment, lock/source revisions, results, failures, artifact hashes, security implications, hardware unknowns and the next ticket. Keep secrets and high-volume build artifacts out of the repository. See [validation](validation.md) and [risks](decisions-and-risks.md).
