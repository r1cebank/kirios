# Decisions, Risks and Deferred Questions

Date: 2026-11-07. This register separates specification decisions from implementation choices still requiring verification. No hardware facts beyond the supplied specification have been inferred.

## Settled by the Specification

- KiriOS / `kirios` / `kirictl` naming; one public management CLI.
- NixOS with integrated Home Manager; separate user-owned runtime preferences.
- Specified initial release branches: NixOS 26.05 and Home Manager release-26.05. Actual commits and compatible Disko revision are not selected yet.
- Stock supported kernel and generic VM defaults before hardware tuning.
- VM-only destructive tests first; live device identity checks and exact typed consent remain mandatory.
- Restricted manifest/static adapter for personal profiles, with explicit conflict handling and no automatic arbitrary Nix import.
- x86_64 gaming only when real graphics capabilities are verified; ARM can use console/fallback graphics.

## Planning Choices

| Choice | Rationale / follow-up |
|---|---|
| Public GitHub repository: r1cebank/kirios | Explicit user request on 2026-11-07; local `main` branch and `origin` established; implementation remains paused |
| Limine installed-system bootloader | Explicit user selection on 2026-11-07; verify locked upstream integration and VM boot/recovery in M0/M1; see [ADR 0008](../adr/0008-limine-bootloader.md) |
| Resume on native x86_64 Linux | Matches the first installer target and permits KVM-based tests; keep macOS as an optional editor |
| M0/M1 first; MVP at M2 | Reconciles initial-deliverable wording with sequential gates; see [ADR 0007](../adr/0007-delivery-boundaries-and-early-cli.md) |
| Small read-only CLI subset before M4 | M1 needs probes and M2 needs doctor/status; expand the same typed Rust CLI later |
| Rust for installer/CLI, QML for shell | Follows the preferred typed runtime in the specification; choose minimal dependencies when implementation starts |
| One unencrypted VM disk layout first | Proves installation before adding encryption/key handling; never imply it is the final laptop recommendation |
| Development trust rooted in ISO-pinned source | Enables reproducible local VM testing without production signing keys; publishing still requires signed release trust |
| Keep original pack and planning separate | Numbered documents define requirements; this folder records execution and status; MANIFEST.json preserves original pack provenance |

## Decisions Due During Implementation

| Decision | Due | Evidence needed / safe default |
|---|---|---|
| Linux distribution, runner ownership and artifact storage | Before M0 | Nix installation and actual KVM/UEFI launch; isolated disposable virtual disks |
| CI runners and workflow permissions | Before remote CI | Repository is now [r1cebank/kirios](https://github.com/r1cebank/kirios); choose isolated native runners and minimal workflow permissions |
| Locked revisions and option compatibility | M0-01 | Fetch/evaluate pinned Nixpkgs/HM/Disko sources; record commit IDs and license/trust |
| InstallPlan identity/fingerprint and privilege transport | Before M1 executor | Negative tests for changed disk/plan, injection and concurrent requests; see [contract gaps](backlog.md) |
| Limine integration and architecture support | M0-02 | Verify locked upstream module/options and both-architecture evaluation; prove x86_64 UEFI boot; no silent bootloader substitution |
| Minimal Btrfs subvolumes and Limine ESP details | M1-04 | Successful install/reboot, previous-generation boot and repair; verify sizing and retention with the selected layout |
| Safe account-password provisioning | M1-05 | Runtime-only secret input, restrictive permissions, no secret/hashes in Nix/store or logs |
| Concrete greeter and VM virtual GPU | M2 | Pinned upstream options and a rendered Hyprland session with recovery |
| Human-editable manifest format | Before M3 | Prefer TOML with preserving edits; supplied YAML is illustrative and must map to a strict versioned schema |
| Personal profile schema and package allowlist | M3 | Architecture checks, path/size/ownership validation and no arbitrary Nix/hook execution |
| age versus SOPS and unlock ownership | M3 optional secret restore | User credential flow, restrictive deployment and failed-unlock tests; skip remains functional |
| Performance orchestration/API | M5 | Reuse upstream power service; custom privileged API only when required and tested |
| Signing and update custody | Before M8 publishing | Owner, trusted public keys, private-key handling and revocation/recovery process |

## Risks and Blockers

| Risk | Impact | Mitigation / trigger |
|---|---|---|
| Current host lacks Nix/QEMU and is ARM macOS | Cannot validate the initial Linux/x86 build or boot path here | User requested continuation on Linux; complete planning and stop |
| Linux CI runners not established | Evaluation/build/VM acceptance checks cannot yet run in CI | Configure native runners during M0; publication alone does not establish build or boot evidence |
| 26.05 support window is short | Release may outlive its supported baseline | Official announcement lists 2026-12-31 end of support; verify status at M0 and before release. If delivery extends beyond support, plan a separately tested lock upgrade/ADR rather than silently changing baseline |
| Published examples are not locked implementations | Wrong option names or schema assumptions could block build/boot | Inspect exact pinned sources and run real native checks |
| Limine integration is selected but unverified | Locked NixOS options, architecture support or recovery behavior may need adaptation | Verify in M0; test ISO-detached boot, previous-generation boot and later LUKS unlock; record gaps before any compatibility change |
| Disk identity/consent race | Wrong target or changed plan could destroy data | Fresh observations, immutable reviewed plan, exclusive executor and fail-closed negative tests |
| Network disappears after formatting | Installation may be incomplete despite successful preflight | Prefetch where feasible, retain rescue/journal and verify post-format recovery; no guaranteed safe resume claim |
| ARM runner or usable VM graphics unavailable | ARM runtime support or Hyprland readiness remains unproven | Separate evaluation/build/boot/graphics evidence; use explicit fallback tiers |
| Profile/desktop/CLI dependencies cross milestones | Duplicate implementations or nonfunctional onboarding | Early shared read-only CLI and integrated HM foundation; later capability expansion remains gated |
| Untrusted profile or privileged control input | Code execution, path traversal or escalation | Restricted manifests, known option/package mappings, narrow authorization and adversarial tests |
| Rebuild/rollback mistaken for data recovery | User files or mutable service data can be lost | Off-host backups and tested recovery; document generation/snapshot limits |
| Hardware power/GPU behavior unknown | Unsafe driver or unsupported control assumptions | Read-only discovery when the relevant stage begins; do not guess from product names |

## Information Needed Later

Do not request these facts to unblock initial VM work.

| Area | Needed facts | Ask when |
|---|---|---|
| XPS | Exact model, PCI/GPU/Wi-Fi IDs, display, storage topology and backup state | Before real power/GPU tests or physical-host qualification |
| ThinkStation | Xeon SKU, exact RTX 6000 generation/PCI ID, firmware/Secure Boot, monitors and disks | Before selecting GPU/compute driver and real gaming tests |
| ARM guest | VM platform, guest firmware, virtual devices and graphics stack | Before ARM runtime validation |
| Personal sync | Repository URL, authentication, intended file ownership and secret provider | M3 integration; use local fixtures first |
| Branding | Logo/assets, licenses and visual preferences | Desktop polish; keep minimal defaults replaceable |
| Physical installation | Verified target identity, off-host backup and explicit erase phrase | Only after VM acceptance and at the actual destructive boundary |

## Upstream Planning References

Consulted on 2026-11-07; these are planning references, not substitutes for locked-source evaluation:

- [NixOS 26.05 announcement](https://nixos.org/blog/announcements/2026/nixos-2605/) confirms release and lists support through 2026-12-31.
- [Home Manager NixOS integration](https://nix-community.github.io/home-manager/nix-flakes/nixos.html) describes the module integration pattern; verify the chosen release branch when locking.
- [Nix remote builds](https://nix.dev/manual/nix/stable/advanced-topics/distributed-builds.html) describes separate supported build platforms, including Linux builders used from macOS.
- [QEMU system introduction](https://www.qemu.org/docs/master/system/introduction.html) lists KVM as Linux acceleration and HVF as macOS acceleration; emulation is a distinct test configuration.
- [Disko quickstart](https://github.com/nix-community/disko/blob/master/docs/quickstart.md) documents explicit target selection and destructive formatting. Use the locked revision during implementation.
