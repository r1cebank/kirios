# ADR 0008: Limine as the Installed-System Bootloader

Date: 2026-11-07. Status: accepted by explicit user choice; implementation and compatibility validation pending.

## Context

The specification requires UEFI boot, a GPT/EFI System Partition layout, previous-generation recovery and disposable-VM boot evidence, but did not select a bootloader. The user selected **Limine** after reviewing that open decision.

## Decision

Use Limine as KiriOS's installed-system bootloader. Retain the existing UEFI-only installation scope. Verify the upstream NixOS integration, exact options, supported target architectures and package version against the Nixpkgs revision locked in M0 before writing configuration.

Prefer the upstream module. If the locked release cannot provide the required integration, record the exact gap and propose a separately reviewed compatibility change. Do not silently substitute another bootloader or claim support based on package availability alone.

The live ISO's boot chain is a separate integration detail: begin from the pinned upstream installation module and inspect its firmware/boot configuration. Record the live image's actual bootloader in build evidence. This decision does not assume that selecting Limine for the installed system automatically changes the ISO's boot chain.

## Implementation and Acceptance

- M0: evaluate the intended configuration for both architectures and prove the minimal x86_64 guest boots through Limine under QEMU/UEFI. Record the locked module/package revision and generated boot configuration.
- M1: size and mount the ESP for the verified integration, install Limine through the validated system configuration and prove boot from the installed disk with the ISO removed. Document repair from the rescue environment.
- Generation recovery: build a second system generation, verify both current and previous entries, and boot the previous generation. Do not infer recovery from a menu screenshot alone. Repeat after the later CLI activation/rollback path exists.
- M1a: verify the selected LUKS/Btrfs layout, initrd and runtime unlock flow with Limine, including wrong-key handling and recovery. No encrypted-boot support claim before these tests pass.
- ARM: evaluate early, then verify native ARM64 UEFI install/reboot on the selected runner. If support is unavailable or untested, keep the ARM gate explicitly blocked or unrun and report the exact limitation.

Secure Boot remains a separate future track requiring a tested trust/signing chain. Choosing Limine does not establish Secure Boot support. Keep at least the required previous working generation available under the eventual retention policy; bootloader recovery does not restore mutable user data.

## Consequences

The bootloader selection is resolved. Option names, ESP sizing/retention, firmware entry handling, architecture compatibility and repair commands remain implementation checks. No boot files, firmware settings or disks are modified during this planning update. Implementation remains paused pending Linux-host continuation.
