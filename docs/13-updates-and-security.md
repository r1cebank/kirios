# 13 · Updates, trust, security and reproducibility

## Release approach

Stable NixOS release, pinned flake inputs, signed ISO checksums, signed binary cache if used, versioned installer/profile schema. Releases should include: `version`, `nixosRelease`, `gitRevision`, `flakeLockHash`, `supportedSystems`, `sha256`, signature, changelog, migration notes, known issues.

## User-facing update journey

`kirictl update check` compares pinned profile revision/release metadata without mutating anything. `kirictl update plan` performs flake evaluation, displays changed inputs + package/service/reboot effects. `kirictl update apply` requires authorization, builds and activates a new generation, then runs post-switch smoke checks. GUI uses same state. Let user postpone major upgrades and reboot.

## Trusted sources

- Release endpoints over TLS, pinned Git commit or signed tag; pin content in `flake.lock`.
- Never pipe fetched arbitrary installer code into `sh`.
- Binary caches require a trusted public key configured from verified release metadata. Never set global `trusted-users = [ "@wheel" ]` casually to make cache setup easier.
- Build ISO only on supported native architecture runner (or explicit tested emulation). Publish SHA256 + signature and verify after writing USB.

## Secret handling

- Nix store contents can be world-readable to local users; no plaintext credentials in Nix expression strings or derivation arguments.
- Password hashes, private keys and Wi-Fi credentials handled through protected runtime inputs/secret manager; inspect logs for leakage.
- Redact package credential URLs and Git auth tokens from logs and diagnostic exports.
- No passwordless sudo for everything; narrowly scope Polkit actions and installer privileges.

## Hardening baseline

- Secure boot optional future track; don't promise working Secure Boot + NVIDIA without tested chain/keys.
- Firewall enabled with minimal openings; SSH disabled by default on consumer desktop unless user requests.
- Flatpak sandbox is useful but not a substitute for application security; keep permissions explicit.
- Vendor GPU drivers are installed only in host-specific modules.
- Use upstream CVE/security advisories and intentional updates; don't indiscriminately track `nixos-unstable` for full system.

## Policy toward third-party inputs

Record maintainer, source URL, tested revision, update cadence and license. Prefer official packages/modules first. Experimental CachyOS kernel integration, Jovian and custom GPU tweaks are optional and require documented impact and recovery plan.

## Backup and rollback scope

- NixOS generations restore system config/package closure if previous generation remains available.
- System rebuild may have persistent side effects and cannot roll back mutated database/home state automatically.
- Maintain off-host backups for home and secrets and separate installer emergency repair instructions.

## Security review checklist

Disk executor injection and TOCTOU; untrusted Git profile hooks; privilege escalation through D-Bus; world-readable token exposure; package cache trust; firmware/driver provenance; path traversal in preference import; accidental log retention of passwords; unsafe rollback assumptions; hidden autostart services.
