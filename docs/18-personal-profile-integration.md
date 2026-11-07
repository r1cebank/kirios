# 18 · How personal Git profiles become active NixOS/Home Manager state

This is a critical integration detail. An ordinary `git clone` in a user's home **does not automatically become a module imported by a pinned NixOS flake**. The implementation must define an explicit, secure bridge.

## Selected v1 design: restricted profile manifest + static distribution adapter

- The distribution flake owns an audited `home-manager` adapter and typed schema.
- A personal Git checkout contains a profile manifest (allowlisted packages/programs, enabled features, theme tokens, managed-file references, machine overlays), plus optional dotfiles assets.
- `kirictl sync` reads the personal manifest, validates schema/paths/size/permissions, previews changes, and creates a **generated declarative overlay** using only known module options and approved file mapping rules.
- `kirictl apply` stages that overlay in an authorized versioned system configuration workspace (never by regex editing `flake.nix`); NixOS integrated Home Manager consumes the generated overlay.
- Store the source revision and content hash with the applied configuration. A subsequent `kirictl sync pull` doesn't apply until approved.
- For dotfiles assets, copy validated contents to a versioned read-only staging location and pass as Nix source paths when building the generation. Avoid symlink/path traversal; managed files must not conflict with runtime preferences.

### Why this approach?

It lets people restore normal settings without giving an arbitrary remote Git repository power to inject unrestricted NixOS module code. Full arbitrary `home.nix` imports are a legitimate expert feature, but **v1 defaults to restricted manifests**.

## Advanced mode: trusted Nix-flake profile (future)

An expert may intentionally trust and pin a personal Git flake with Home Manager modules. This requires separate explicit review/authorization and a clear trust warning. Private Git auth must not be baked into system/store; prefer staging authenticated source outside source derivations or a credential helper that does not leak secrets.

## Applying profiles to NixOS

1. **Distribution policy layer:** immutable distro module defaults and allowed settings.
2. **Generated profile layer:** named host preference/profile overlay from validated personal manifest.
3. **User live layer:** mutable runtime preferences managed solely by Quickshell/`kirictl widgets`; imported/exported intentionally, not regenerated on every switch.

Precedence is explicit: system safety policy > hardware capability > validated personal declared settings > default. Live UI preferences can override desktop appearance until export/import or reset, but cannot change disk or driver safety policy.

## Multi-user behavior

A trusted system administrator approves which profile is bound to which local user. Each user's Git credentials and XDG preferences remain private. Multiple users must not accidentally share token files, preference DBs, or system services. System-level changes remain admin-authorized.

## Acceptance scenario

1. Fresh install with no user Git: functional default shell.
2. User clones personal repo containing terminal/keybindings/theme declarations; preview shows files and package changes.
3. Apply safely builds system with integrated HM and activates it; `kirictl status` shows source revision.
4. User changes widget layout live; `nixos-rebuild switch` does not rewrite layout.
5. Remote profile updates causing conflicts are visible and cannot silently wipe local edits.
6. Same profile on ARM64 excludes x86-only binaries and has appropriate VM fallback.

## Agent implementation checklist

- Explicit data model and schema for manifest; no arbitrary Nix expression fragments accepted from restricted mode.
- Path canonicalization and file allowlist for profile assets.
- Transactional profile staging and safe restoration on activation failure.
- Lock/profile revision recorded and testable.
- Clear error diagnostics for unsupported package/features and duplicate file ownership.
