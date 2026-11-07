# 04 · Partitioning, encryption, boot and recovery

## v1 target disk layout (UEFI only)

Single selected disk, GPT, EFI System Partition (size selected after bootloader validation, initially >= 1 GiB), optional LUKS2 container, Btrfs root with `@root`, `@home`, `@nix`, `@persist` or an intentionally simpler verified subvolume scheme. Mount with supported compression settings; configure fstrim service on SSD/NVMe. Disko defines repeatable layout; **Disko's destructive modes erase partitions**.

### Design cautions

- EFI partition is not Btrfs snapshot-protected by default.
- NixOS generations roll back system store/configuration, **not necessarily mutable `/home` data, databases or EFI content**.
- Btrfs snapshots help recover accidental file changes but need separate off-device backups, plus snapshots may include sensitive files.
- Don't combine LUKS disk unlock keys and secrets in a public flake or ISO; prompt securely at install time.
- Lock/unlock semantics of systemd initrd changed in NixOS 26.05; validate LUKS mapper/root mount values against release notes.
- Firmware/UEFI bugs and NVIDIA may complicate hibernation; skip hibernation first, test suspend carefully.

## Suggested defaults

| Target | Disk | Encryption | Swap |
|---|---|---|---|
| XPS | Single internal SSD selected explicitly | LUKS2 suggested | ZRAM initially; optional persistent swap sized for real hibernation needs |
| ThinkStation | Selected OS disk, never data disks | User choice, LUKS2 suggested if sensitive | Minimal/none initially due to 256GB RAM |
| ARM VM | Disposable virtual disk | Optional in test builds | Minimal swap or ZRAM, constrained to VM memory |

## Recovery routes

1. Greeter/TTY safe session independent of Quickshell.
2. Boot menu with previous NixOS generation.
3. Installer USB > recovery: open LUKS, mount known Btrfs layout, `nixos-enter`/repair options, roll back or reinstall bootloader.
4. Data restoration from separate backup; document exactly what is and isn't captured.
5. `kirictl doctor` system checks before suggesting any write operation.

## Failure taxonomy

- Evaluation/build failure before switching: old generation remains active.
- Switch failure at runtime: return status; user can reselect previous generation. **`nixos-rebuild switch` is not a universal atomic rollback of mutable side effects.**
- Boot failure: older boot generation via boot menu.
- Disk failure: backups/reinstall; snapshots on the same failing disk do not help.
- Sync conflict: preserve both copies, do not auto-delete personal files.

## Backups and retention

- Back up Home Manager repo, runtime settings export, encryption recovery keys and mutable home data off-host.
- Nix store garbage collection may remove old generations; define retention and require backup verification before aggressive cleanup.
- Check Btrfs health with supported filesystem tools; no routine scrub on unvalidated disk layout without suitable schedules and logs.
- Test recovery in a VM as a release gate.
