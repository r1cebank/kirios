# Validation and Evidence Plan

Date: 2026-11-07. Commands in this document are **planned checks**, not execution results. The flake, checks and VM harness do not exist yet. Only the [planning report](../../tests/reports/2026-11-07-planning.md) records observations from this session.

## Environments

| Environment | Responsibility |
|---|---|
| Current macOS arm64 workspace | Documentation, source editing and portable tests only where available; no Linux build or boot claims |
| Native x86_64 Linux + KVM + OVMF | Primary system/ISO builds, fast disposable QEMU tests and M0-M2 acceptance |
| Native aarch64 Linux + UEFI VM support | ARM system/ISO builds and console install/reboot; qualify guest graphics separately |
| XPS / ThinkStation | Read-only identification and later real GPU/power/device qualification; physical installs only after VM gates, backup and explicit typed target consent |

A remote Nix builder can build Linux artifacts for a macOS editor. It does not by itself provide the test orchestration, KVM access, screenshots or installed-disk reboot evidence. Record both builder and test-runner environments.

## Readiness Checks on Linux

Record these read-only checks when resuming. Failures identify missing prerequisites; do not claim these ran on the planning host.

```sh
uname -srm
git status --short --branch
nix --version
qemu-system-x86_64 --version
test -r /dev/kvm && test -w /dev/kvm
df -h .
```

Also record CPU virtualization support, firmware package/path/version and actual VM acceleration. The `/dev/kvm` access check alone does not prove a VM can start. After the dev shell exists, use its pinned formatting, shell and contract-check tools rather than assuming global installations.

## Planned M0 Checks

After the real flake exists, run the following on the appropriate Linux environment and save stdout, stderr and exit status. The attribute names are required/planned interfaces, not existing outputs.

```sh
nix flake metadata --json
nix flake check --no-build
nix eval --raw .#nixosConfigurations.vm-test.config.system.build.toplevel.drvPath
nix eval --raw .#nixosConfigurations.arm-vm.config.system.build.toplevel.drvPath
nix eval --raw .#nixosConfigurations.xps.config.system.build.toplevel.drvPath
nix eval --raw .#nixosConfigurations.thinkstation.config.system.build.toplevel.drvPath
nix flake check
```

Explicitly evaluate all four hosts; do not assume a platform-filtered check covers both architectures. Run native build checks on their respective runners. Add assertion tests for unsupported feature combinations and inspect the ARM dependency graph for unintended x86/vendor packages.

The M0 VM check must boot the minimal x86_64 system through the selected Limine bootloader under UEFI, reach login and prove networking. Verify locked upstream module/options and architecture support first; see [ADR 0008](../adr/0008-limine-bootloader.md). Use the NixOS test framework where it fits; ordinary module tests supplement, rather than replace, the ISO installation test below.

## Planned M1 ISO and Installation Checks

```sh
nix build .#packages.x86_64-linux.iso
```

Build `.#packages.aarch64-linux.iso` separately on its native Linux builder when available. Record the output store path and ISO SHA-256, lockfile hash and source revision. Define the exact QEMU command only after firmware, disk layout and virtual devices have been selected and pinned; do not copy an untested host-specific firmware path into a runbook.

The harness must:

1. Create a fresh sparse target disk and a separate sentinel disk with stable distinct virtual serials; attach no physical host devices. Preserve a pristine baseline for negative tests.
2. Boot the actual built ISO with fresh per-test UEFI variable storage, network and serial capture. Exercise the real frontend-to-executor path and type the guest's exact erase phrase after reviewing the plan.
3. Verify the selected virtual disk is installed and the sentinel remains byte-identical. For preflight/confirmation failures, require byte-identical target and sentinel images after guest shutdown.
4. Cover missing network, invalid source/manifest, missing/mounted/live-media disk, incorrect confirmation, disk replacement and plan mutation. Prevent writes after any failed guard.
5. Interrupt after formatting and during installation. Confirm no automatic reformat, accurately reported partial state, and only verified post-format recovery.
6. Shut down, detach the ISO and boot the installed disk with the same firmware state. Log in and verify bootloader, root mounts, source revision, account and networking; reboot offline too.
7. Scan exported journal/logs and relevant configuration/store outputs using synthetic secret canaries. Inspect account setup and runtime key paths for exposure; do not log real credentials to demonstrate redaction.

Record the live ISO's actual boot chain separately from the installed Limine configuration. After installing, create a second generation, verify both entries and boot the previous generation. Test rescue repair; repeat generation recovery after the later CLI activation path exists. In M1a also verify LUKS unlock and wrong-key handling with this bootloader/layout. Secure Boot is not accepted without a separately tested signing/trust chain.

Guest filesystem/login evidence plus ISO-detached reboot is mandatory. An ordinary prebuilt NixOS VM test is not proof that the installer works. A green build cannot substitute for missing E2E execution.

## Planned M2 and Later Checks

| Gate | Positive path | Failure/recovery path |
|---|---|---|
| Desktop MVP | Fresh install, greeter -> Hyprland/UWSM, rendered panel, terminal/app launch, doctor, offline login | Kill shell, restart service, corrupt preferences, rebuild, reboot; verify TTY and fallback, no login loop |
| Home/profile | Two installations, pinned profile/lock hashes and declared state comparison | Existing-file conflict, hostile path/symlink, untrusted hook/Nix, wrong key and offline fetch preserve working state |
| Apply/rollback | Intent diff -> authorized build/activation -> status -> previous generation | Denied authorization, concurrent apply, failed build/switch and reboot recovery; distinguish system rollback from mutable data restoration |
| Performance | Requested/effective modes agree, precedence and supported profiles tested | Unsupported hardware, missing sensors, rejected request, backend/game crash; measured polling and restoration |
| Developer | Terminal and GUI Node, desktop-launched VS Code MCP JSON-RPC fixture, NVM, Python/Cargo/container | PATH mismatch, missing loader/libraries and sandbox paths produce useful diagnostics |
| Gaming/hardware | Real supported GPU, Vulkan including 32-bit path, named Proton title, GameMode exit | Failed console session returns to greeter; suspend/driver failures retain recovery access |
| Release | Signed ISO/metadata verified, fresh install, optional restore and recovery | Invalid signature, failed update, protected generations and off-host data restore |

For desktop validation capture an actual guest screenshot and assert session/service state; a running process alone does not show a usable desktop. ARM fallback tests remain separate from x86 Hyprland acceptance. Record exact real/mock and supported/unsupported boundaries.

## Evidence Requirements

Use the [report convention](../../tests/reports/README.md). Each milestone report includes source/lock revisions, runner and tool versions, exact commands/statuses, artifact hashes/paths, observed behavior, failures/not-run checks, security findings, hardware unknowns and the next ticket. Keep large artifacts and secrets outside Git; link sanitized logs with retention/access details.

CI should initially run formatting, contract tests, both-architecture evaluation and native build checks. Schedule expensive ISO/E2E jobs on isolated trusted Linux runners. Lack of a runner is a blocker to that gate, not a reason to weaken its definition.
