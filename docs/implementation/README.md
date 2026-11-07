# KiriOS Implementation Planning

Last updated: 2026-11-07. Status: planning complete; implementation not started. The user requested a stop after planning and continuation on a Linux host.

## Read and Resume

1. Read [AGENTS.md](../../AGENTS.md), then [the implementation plan](plan.md).
2. Check the Linux prerequisites below and [the open decisions](decisions-and-risks.md).
3. Start with M0 in [the backlog](backlog.md). Work through one milestone at a time.
4. Follow [the validation guide](validation.md); save actual results in [tests/reports](../../tests/reports/README.md).
5. Update this status and the backlog after each accepted gate. A blocked gate remains incomplete.

## Current State

| Area | Observed state |
|---|---|
| Source | AGENTS.md, README, 20 numbered specifications, four prompts, six contract/example files, original manifest |
| Implementation | No flake, lockfile, application source, CI, ISO, or VM tests yet |
| Version control | Git initialized on `main`; public repository [r1cebank/kirios](https://github.com/r1cebank/kirios) configured as `origin` on 2026-11-07 |
| Planning host | Darwin 27.0.0, arm64; an editing environment, not a verified Linux builder |
| Tools | Git, SSH, Rust/Cargo, Python and Node found; Nix, QEMU, shellcheck, shfmt and nixfmt not found on PATH |
| Evidence | [Planning report](../../tests/reports/2026-11-07-planning.md); no Nix, ISO, boot or installation passes claimed |
| Next action | Establish the Linux build/test environment, then implement M0 |
| Bootloader | Limine selected for installed systems; [ADR 0008](../adr/0008-limine-bootloader.md); integration and boot tests pending |

No disks were changed, packages installed, or implementation milestones completed during planning. After planning, the user requested public GitHub publication; Git tracking and the remote were established separately. The original planning report remains a historical record of the environment before that setup.

## Linux Host Readiness

Prefer a native **x86_64 Linux** machine for M0-M2. It may run an existing Linux distribution with Nix; it does not need to run KiriOS. A macOS editor with a remote Linux builder is possible, but a Linux VM test runner is also required. An ARM64 Linux host covers ARM builds and tests; it does not replace native x86_64 validation without explicitly tested emulation.

- Provide Nix with flakes support, Git, outbound access to the chosen sources/caches, and sufficient free build storage.
- Provide QEMU x86_64, OVMF/UEFI firmware, and working KVM access for the test user. If Linux itself runs in a VM, verify nested virtualization before scheduling E2E tests.
- Start with a planning budget of 8 CPU threads, 16-32 GiB RAM and 100 GiB free storage; these are estimates to revisit after the first build, not measured minimums.
- Keep the source checkout and build artifacts on the Linux filesystem. Use only newly created virtual disks; do not pass host block devices through to the guest.
- Record OS, architecture, Nix/QEMU/firmware versions, virtualization access and storage before the first build. Pin test dependencies through the flake where practical.
- Inspect the new checkout for local instructions and changes. Establish Git tracking locally if it is still absent; repository hosting and pushing remain separate actions.
- Keep any SSH builder credentials outside Git, the flake and build logs. Use a narrowly scoped build account if a remote builder is configured.

Exact provisioning commands depend on the Linux distribution. No toolchain installation or host configuration was performed in this planning session. See [validation](validation.md) for proposed readiness checks.

## Suggested Resume Request

> Read AGENTS.md and docs/implementation/README.md, plan.md, backlog.md, validation.md, decisions-and-risks.md, and docs/adr/0008-limine-bootloader.md. Inspect this Linux host and the working tree. Implement M0 only, using the specified 26.05 baseline unless source verification requires a documented change. Use the user-selected Limine bootloader for installed systems and verify its locked upstream integration. Evaluate all four hosts, build and boot vm-test under QEMU/UEFI on disposable storage, and record exact evidence. If M0 passes, report it and identify M1 as the next milestone. Do not touch physical disks or start later features while a gate is failing.

## Planning Documents

| Document | Responsibility |
|---|---|
| [Plan](plan.md) | Product boundaries, architecture and milestone gates |
| [Backlog](backlog.md) | Ordered implementation slices and acceptance criteria |
| [Validation](validation.md) | Environments, planned checks, failure tests and report format |
| [Decisions and risks](decisions-and-risks.md) | Defaults, unresolved choices and when information is needed |
| [ADR 0007](../adr/0007-delivery-boundaries-and-early-cli.md) | Reconciliation of sequencing conflicts in the original pack |
| [ADR 0008](../adr/0008-limine-bootloader.md) | User-selected Limine bootloader and required integration/recovery tests |

The original [delivery roadmap](../15-delivery-roadmap.md) retains the canonical M0-M8 ticket IDs. Update implementation status here and in the backlog, and update its checkboxes only when the corresponding acceptance evidence exists. Do not interpret old agent prompts as authorization to exceed the current request.
