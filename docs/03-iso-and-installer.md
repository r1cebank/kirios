# 03 · Custom ISO and quick-install specification

## Experience goal

USB/VM boot -> automatic locale/welcome (TUI) -> wired/Wi-Fi/time check -> license/disclaimer -> choose profile + disk + encryption + user + hostname -> **review immutable install plan** -> install -> reboot -> greeter -> ready-to-use desktop + optional Welcome.

## Two image products

- `kirios-installer-x86_64.iso`: XPS, ThinkStation, x86 VM; generic kernel/drivers on live image, no workstation-specific closed-source GPU assumptions.
- `kirios-installer-aarch64.iso`: ARM64 VM with UEFI boot. Test exact firmware/virtual hardware and accelerator path; do not claim all macOS VM stacks support graphics. VM first phase may use console-only install.

The image includes Nix CLI, networking/Wi-Fi tools, SSH/recovery utilities, TUI binary, trusted release metadata/keys, deterministic install configuration generator, Disko integration and basic troubleshooting commands. The **target desktop** is installed through pinned remote sources/cache, not baked into the live shell by default. Show network requirements in UI.

## Proposed UI screens and validation

| Screen | Fields | Guardrails |
|---|---|---|
| Welcome | Install, diagnostics, recovery shell | Never auto-trigger destructive stage |
| Network | Wired, Wi-Fi SSID/PSK, proxy | Connectivity and NTP tests; never log PSK |
| Platform | Architecture, UEFI, GPU, battery, VM detection | Allow manual profile override; confidence badge in UI only |
| Disk | Stable disk ID, make/model, size, partitions | Show existing filesystem/mountpoints; exclude installer USB and mounted disks; fail closed on ambiguity |
| Layout | Fresh single disk, LUKS2 optional, Btrfs | v1 no in-place resize or dual boot; unsupported cases block |
| Account | Name, hostname, locale, keyboard, timezone | Password via no-echo TTY; no password in Nix store |
| Features | desktop, dev, gaming (if supported), initial theme | ARM64 gaming disabled with explanation |
| Review | Exact disk serial/size, data-loss warning, profile, release rev | Require typing `ERASE <disk-id>` for destructive operation |
| Progress | Stage, bytes, logs, retries | Distinguish recoverable fetch failure from partial disk write |
| Finish | Success, next boot, saved non-secret report | Never include passwords/tokens in report |

## Execution phases

1. **Preflight, no disk writes:** UEFI/arch, real target identity, AC power guidance, clock, DNS/TLS, caches accessible, release integrity verification, required payload prefetch where feasible, target flake evaluation, key availability.
2. **Plan generation:** typed `InstallPlan v1` JSON (see contracts), validate schema and host capabilities; provide `--dry-run` output and full disk operation diff.
3. **Destructive stage:** explicit interactive confirmation with disk ID, Disko format/mount. No generic `curl | sh`. Journal sensitive-free results.
4. **System stage:** `nixos-install` against generated configuration, generated host hardware configuration and proper ESP/LUKS mounts, create account securely, install bootloader, validate closure/ESP.
5. **Completion:** unmount, save non-secret installer report and pinned source revision on target, reboot only after user approval.

## Disk path protection

Do not concatenate an untrusted disk path into shell. Use stable `/dev/disk/by-id/...` links, resolve symlinks, verify `lsblk` major:minor, `udevadm` identity and mount relationships immediately before execution. Abort if the disk disappeared or changed identity. Recheck after confirmation. Never allow `/dev/loop*`, installer medium, or mounted host volumes.

## Custom ISO implementation strategy

- Use NixOS's installation ISO module as the base; add `installer` service or autostart only on a safe live TTY. The normal rescue shell remains available.
- Installer frontend and executor communicate over a typed serialized plan (JSON/IPC); executor owns all privileged operations.
- Install source must be a **pinned Git revision or signed release manifest**, not a floating branch that can change while installation is running.
- Optionally support offline artifact cache later; network-enabled install is core v1.
- Persist progress in RAM and optional destination `/var/lib/kirios/install-journal.json` only after filesystem mounts; do not misrepresent interrupted formatting as safe resume.
- Include “Install in VM” workflow with sparse virtual disk and OVMF UEFI; emit artifact hashes/manifest.

## Command contracts (project-defined, not upstream commands)

```text
project-install --check-network
project-install --probe-hardware --json
project-install plan --input answers.json --output plan.json
project-install validate plan.json
project-install execute plan.json       # always confirms target; no implicit --yes on physical devices
project-install diagnose --export report.json
```

## Error conditions

- No network before partition: return to Network page.
- Corrupt/untrusted release: abort, keep disks untouched.
- Disk operation failure: stop, inspect state; never automatically rerun format.
- `nixos-install` failure after partition: enable repair/resume post-stage only after verifying mount+disk UUID+flake revision.
- No graphics after reboot: provide TTY shell, known-good system generation and documented safe greeter fallback.

## Acceptance

VM tests for wrong-disk rejection, offline preflight, incorrect confirmation, UEFI boot, LUKS unlock, fresh install and reboot into greeter. On ARM64, validate ISO boot and command-line install before graphical claims.

**Upstream:** https://nixos.org/manual/nixos/stable/ ; https://github.com/nix-community/disko ; https://github.com/nix-community/nixos-anywhere
