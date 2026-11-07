# KiriOS: Agent Build Pack

**Status:** Implementation specification, not a working distribution.  
**Baseline:** NixOS 26.05, pinned via `flake.lock` at implementation time.  
**Initial specification:** 2026-10-12.  
**KiriOS branding revision:** 2026-10-13.  
**Project name:** **KiriOS** (canonical); **namespace:** `kirios`; **management CLI:** `kirictl`.  
**Target architectures:** `x86_64-linux` (Dell XPS, ThinkStation) and `aarch64-linux` (macOS-hosted Linux VM).
**Public repository:** [r1cebank/kirios](https://github.com/r1cebank/kirios).

## Implementation planning

Planning is documented as of **2026-11-07**. Implementation has not started; resume on a Linux host after the readiness checks below. The first implementation scope is M0, followed by M1; the usable desktop MVP is complete only after M2 passes.

- [Planning index and Linux handoff](docs/implementation/README.md)
- [Implementation plan](docs/implementation/plan.md)
- [Ordered backlog](docs/implementation/backlog.md)
- [Validation and evidence](docs/implementation/validation.md)
- [Decisions, risks, and deferred questions](docs/implementation/decisions-and-risks.md)
- [Planning baseline report](tests/reports/2026-11-07-planning.md)

The numbered specifications below remain the product baseline. The planning documents translate them into executable work; [ADR 0007](docs/adr/0007-delivery-boundaries-and-early-cli.md) records the MVP boundary and early CLI dependencies. `MANIFEST.json` is the original specification-pack inventory, not a live inventory of this evolving workspace.

## Project identity

**KiriOS** is the operating system and distribution brand. Its internal identifier is `kirios` (lowercase). Components are **Kiri Setup**, **Kiri Desktop**, **Kiri Shell**, **Kiri Welcome**, **Kiri Sync**, and **Kiri Performance Center**. The sole project management CLI is `kirictl`. See [`docs/19-brand-and-naming.md`](docs/19-brand-and-naming.md).

## Mission

Build a reproducible, approachable NixOS distribution with a fast **network-assisted ISO installation**, first-boot **home/environment sync**, polished **Hyprland + Quickshell** desktop, **kirictl** management CLI, safe **Performance Center** widget, and optional **Steam/Proton/Gamescope gaming session**. Optimize *measured performance and efficiency*, not benchmark theater.

This ZIP contains specifications, contracts, a proposed backlog and agent prompts. It deliberately does **not** claim to contain a functioning ISO, tested Nix modules, production signing keys, or a completed application.

## Read first, in order

1. [`AGENTS.md`](AGENTS.md): mandatory agent rules and definition of done.
2. [`docs/00-product-and-decisions.md`](docs/00-product-and-decisions.md): goals, non-goals, decisions, unresolved questions.
3. [`docs/01-architecture.md`](docs/01-architecture.md): system boundaries and configuration ownership.
4. [`docs/02-flake-and-repository.md`](docs/02-flake-and-repository.md): implementable repository layout and Nix module conventions.
5. [`docs/15-delivery-roadmap.md`](docs/15-delivery-roadmap.md): milestones, ordered tickets, and gates.
6. [`docs/19-brand-and-naming.md`](docs/19-brand-and-naming.md): canonical identity and naming conventions.
7. [`agent-prompts/00-kickoff.md`](agent-prompts/00-kickoff.md): exact kickoff request to paste into your coding agent.

## Spec index

| Area | Documents |
|---|---|
| Installation | `docs/03-iso-and-installer.md`, `docs/04-disk-and-recovery.md` |
| Hardware | `docs/05-hardware-and-platforms.md` |
| Desktop | `docs/06-desktop-and-theme.md` |
| User sync | `docs/07-home-sync-and-secrets.md`, `docs/18-personal-profile-integration.md` |
| CLI | `docs/08-kirictl-cli-and-config.md` |
| Performance | `docs/09-performance-center.md`, `docs/10-performance-engine.md` |
| Gaming | `docs/11-gaming-stack.md` |
| Developer compatibility | `docs/12-developer-experience.md` |
| Operations/security | `docs/13-updates-and-security.md` |
| Validation | `docs/14-test-plan.md` |
| Roadmap and agent instructions | `docs/15-delivery-roadmap.md`, `docs/16-agent-workflow.md`, `agent-prompts/` |
| Research | `docs/17-sources-and-verification.md` |
| Branding and namespaces | `docs/19-brand-and-naming.md` |
| Examples | `contracts/` |

## First implementation target

Ship a **VM-installable x86_64 MVP** before touching the XPS/ThinkStation disks. The MVP ISO must boot, connect to a network, guide a disk install using a verified target, reboot into a working Hyprland session, launch a minimal Quickshell panel, and run a `kirictl doctor` check. Add full sync and performance policy next. Never use physical hardware as the first disk-destructive test.

## Critical decisions

- NixOS owns the boot, drivers, system services and global packages; **integrated Home Manager** owns user packages and managed dotfiles; **XDG mutable files** own live widget preferences. No duplicate ownership of the same file.
- The CLI is **`kirictl`**, not `homectl`: `homectl` already belongs to systemd-homed.
- A single pinned flake exposes three host variants and separate ISO outputs; architecture-gated features prevent evaluation errors on ARM64.
- Start with stock NixOS kernels, standard binary caches, adaptive power management and measured benchmarks. CachyOS patches, x86-v4 recompilation, alternative schedulers and experimental graphics are **optional experiments**, not baseline requirements.
- Steam/Proton and Gamescope are x86_64-only in this scope; ARM64 VM focuses on installer validation, terminal/development and a graphical session **only if the VM graphics stack supports it**.
- Secure, recoverable installation and reproducible rollback are higher priority than zero-click disk wiping or visually elaborate setup.

## Success metrics

- A second installation reproduces installed packages, NixOS services, declared home programs and desktop defaults from the same revision.
- A terminal, VS Code launched from desktop, and an MCP stdio subprocess can all run a tested Node executable; project-local NVM behavior is explicitly validated.
- Steam launches, Vulkan works, a Proton game starts and GameMode exits cleanly on supported x86_64 hardware.
- Power changes remain reversible and do not stack conflicting governors/managers.
- Failed `kirictl apply` leaves the previous working generation selectable.

## Handoff

Give the entire ZIP to an agent. Say: **“Read `AGENTS.md`, `README.md`, and the complete docs index. Implement only Milestone 0 and Milestone 1 first. Preserve the contracts and record deviations as ADRs.”**
