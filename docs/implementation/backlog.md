# Ordered Implementation Backlog

Date: 2026-11-07. Every item is not started. IDs refer to [the original roadmap](../15-delivery-roadmap.md); substeps below elaborate those tickets. Check an item only after its acceptance evidence is linked from a report.

## M0: Foundation

Prerequisite: [Linux host readiness](README.md) recorded. No physical disk access is needed.

- [ ] M0-01: Inspect local instructions and tree, initialize local Git tracking if absent, and add narrowly scoped ignores for build/VM artifacts. Verify the specified release branches, compatible Home Manager and Disko commits; create `flake.nix` and a real `flake.lock`. Document input sources, licenses and update policy.
- [ ] M0-02: Implement `modules/core/`, minimal VM hardware and `hosts/{vm-test,arm-vm,xps,thinkstation}` composition. Verify the user-selected Limine integration against locked upstream NixOS sources and both architectures; provide Limine UEFI boot, network, safe local user setup and recovery console. See [ADR 0008](../adr/0008-limine-bootloader.md). Physical hosts remain explicitly unqualified templates; no guessed UUIDs or GPU IDs.
- [ ] M0-03: Add architecture/capability assertions. Test that gaming and proprietary vendor selections fail safely on ARM and are disabled by default for unknown hardware. Keep the stock kernel baseline.
- [ ] M0-04: Add dev shells, pinned formatting/check tooling and native Linux CI. Evaluate all host outputs, build the current architecture's checks, and create an x86_64 QEMU/UEFI VM boot check. Do not enable privileged tests on untrusted contributions without isolation.
- [ ] M0-05: Record configuration ownership, unresolved hardware, security review and evidence. Verify that public sources, generated configuration and logs contain no credentials or local private source paths. Save a boot/network report.

Planned paths: `flake.nix`, `flake.lock`, `modules/core/`, `modules/hardware/`, `hosts/`, `tests/eval/`, `tests/vm/`, `.github/workflows/`. Add Home Manager to the module graph here; user-program and desktop defaults arrive in M2.

Gate: all named hosts evaluate, relevant native checks pass, and a minimal x86_64 guest boots via UEFI and has network. A successful Nix evaluation alone is insufficient. Next: M1-01.

## M1: Guarded Network Installer

- [ ] M1-01: Build an x86_64 ISO from the locked upstream installation module. Package network/clock diagnostics, rescue console, installer and source metadata. Boot the actual ISO under UEFI before implementing writes.
- [ ] M1-02: Implement no-write probe/preflight and minimal Kiri Setup TUI. Cover wired networking, runtime Wi-Fi credentials, clock/DNS/TLS, source/cache access and safe disk enumeration. Implement the small read-only `kirictl hardware probe` subset needed for diagnostics using the same typed probe model.
- [ ] M1-03: Harden and test `InstallPlan` and confirmation protocol before the executor. Add valid/invalid fixtures; define source/hash validation, field constraints, architecture/feature rules and a fresh disk observation. Bind consent to immutable plan content. Freeze the fixture/schema relationship before UI integration.
- [ ] M1-04: Implement one reviewed GPT + ESP + unencrypted Btrfs layout through pinned Disko. First dry-run the operation preview, then test only inside a disposable VM. Exclude installer/live media, mounted/in-use devices and ambiguous targets. Repeat identity checks after typed confirmation immediately before writes.
- [ ] M1-05: Build/fetch the pinned target before destruction where feasible, install generated and validated target/hardware configuration, set account credentials through protected runtime input, install Limine and save non-secret source metadata. Verify closure/mount/ESP state, then offer reboot. Prove ISO-detached boot, current/previous generation boot and rescue repair. Never encode passwords/hashes as Nix strings.
- [ ] M1-06: Add redacted stage journal and error recovery. Test wrong confirmation, replaced disk/plan, network loss, corrupt source, executor interruption and failed installation. Never retry formatting automatically; post-format resume must independently verify disk/filesystem identity, mounts and source revision.
- [ ] M1-07: After x86 passes, build the ARM ISO on native ARM64 Linux and test UEFI console install/reboot. If unavailable, preserve the output/evaluation work and explicitly mark build/boot tests not run.

Planned paths: `installer/{iso,tui,executor,profiles}/`, `contracts/`, `tools/kirictl/`, `tests/{unit,integration,vm}/`. The executor is an internal component, not a second public management CLI. Do not implement the illustrative `project-install` command names as a new public interface.

Gate: boot the built ISO, complete a confirmed install into a virtual target disk, remove the ISO, boot the installed system, log in and verify network/offline reboot. Record disk-guard failures and redaction checks. Next: M2-01.

### Contract Work Required Before M1 Writes

| Gap in supplied examples | Required resolution |
|---|---|
| Model/serial are optional; no major:minor field | Specify required executor observations separately from portable plan data; fail closed when identity cannot be established. Give test disks stable virtual serials. A by-id string alone is not consent. |
| Revision/hash constraints check length only | Define canonical full source revision and lock-content digest formats, validate actual values, and tie installed content to the reviewed source. |
| Schema permits unsafe combinations | Add semantic validation for host/arch, supported features, disk usage/size, locale/timezone, username and hostname. Syntax validation does not replace capability or device checks. |
| No binding between preview and execution | Freeze/revalidate the plan and retain a fingerprint plus fresh disk observation. Reject changes after review; no reusable consent token for altered plans. |
| Trust bootstrap not specified | Use a trusted source revision/hash embedded in the development ISO initially. Reject invalid signed manifests if supplied; do not require production signing keys to test a locally pinned development ISO. Define signed release metadata before publication. |
| Dry-run could accidentally invoke Disko | Specify no-write guarantees and test host-side virtual-image hashes plus guest device observations. Preflight may access network/store but must not write the target. |
| Executor privilege boundary is only prose | Define allowed requests/arguments, caller authorization, exclusive execution, restricted paths and redacted logging. Ensure no arbitrary file/shell operation is accepted. |

Record material schema/protocol decisions as ADRs and update affected contracts together. All of this is planned work; existing contracts have not been silently changed.

### M1a: Encryption Follow-Up

- [ ] After the unencrypted gate, add LUKS2 using runtime key input and the locked initrd options. Cover unlock, wrong key, reboot and recovery in a VM; inspect logs/store for leakage.
- [ ] Document recovery-key handling and what generations/snapshots do not restore. Keep encryption unavailable in the UI until the path passes.

M1a may follow M1 acceptance without delaying the initial unencrypted VM experiment; it is required before claiming encrypted-install support.

## M2: Desktop MVP

- [ ] M2-01: Add UWSM/Hyprland, Xwayland, portals, audio, clipboard, screenshots and lock/idle, verifying option names against the lock. Avoid duplicate session management.
- [ ] M2-02: Add greeter, terminal, file manager/browser and a reliable TTY/session fallback. Verify actual virtual GPU support before declaring the Hyprland gate.
- [ ] M2-03: Package minimal Kiri Shell top bar and launcher; attach one user service to graphical session lifetime. Package licensed assets locally.
- [ ] M2-04: Define defaults and schema for mutable XDG preferences, atomic updates and corrupt-file fallback. Test persistence after shell restart, reboot and integrated rebuild.
- [ ] M2-05: Add minimal Kiri Welcome with skip/offline flow, plus read-only `kirictl status` and `doctor` with the existing JSON/error/exit-code contracts. Do not stub later commands as successful operations.

Planned paths: `modules/desktop/`, `home/{common,desktop}/`, `desktop/{hyprland,quickshell,theme}/`, `tools/kirictl/`, `tests/vm/`.

Gate: a fresh ISO install boots a working Hyprland/UWSM session and Quickshell panel without Git or network dependence. Verify app launch, doctor, audio where available, preference persistence and TTY recovery. Only then call the result an MVP.

## Later Milestones

Detailed contracts should be refined at each milestone boundary after prior gates pass. Do not begin these subsystems during M0-M2.

| Stage / original tickets | Implementation focus | Required evidence |
|---|---|---|
| M3-01..05 | Extend existing integrated HM with restricted manifest/adapter, private Git credentials, conflict-preserving backups, host overlays and optional age/SOPS | Hostile-path tests, no arbitrary Nix/hooks, identical declared state on two installs, original-byte preservation and no secret leaks |
| M4-01..05 | Expand early Rust CLI into full management, preserving JSON/exit codes; validated intents, diff/apply, authorization, serialized activation, generations and rollback | Contract tests plus actual VM activation/failure/rollback and previous-generation boot |
| M5-01..06 | Capability probes, nullable telemetry, policy precedence, reversible upstream mode requests and Performance Center | Mock/real labels, permission denial, missing sensors, service/game crash restoration and measured polling overhead |
| M6-01..05 | x86 GPU-gated Steam/Vulkan/GameMode, optional Gamescope; stable/project/compatibility developer runtimes | Real compatible Proton title and driver details; GUI-launched VS Code/MCP Node fixture, NVM, Python/Cargo/container smoke |
| M7-01..05 | Exact hardware configuration, suspend/devices, ARM graphics tier, stock baseline and optional reversible experiments | Physical host matrix, backup/recovery drills, repeatable measurements and documented regressions |
| M8-01..05 | Signed artifacts/metadata, native release CI, optional builders/cache, updates/retention and recovery docs | Verified release install/restore/rollback, support status, signing custody and published limitations |

## Updating Status

For every completed ticket, add the report path beside its checkbox and update [the handoff](README.md). Keep failed or unrun checks explicit. Update the original roadmap's matching item only when its scope is fulfilled; early subsets of M4 do not complete M4 tickets. Keep hardware facts and design changes in [the decision register](decisions-and-risks.md) and ADRs.
