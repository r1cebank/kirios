# 07 · Home Manager, Git sync, secrets and machine overlays

## Three kinds of home state

1. **Declarative (`home/` repo):** Home Manager app packages, shell programs, templates, keybindings and default Quickshell config. Apply via NixOS-integrated Home Manager generation.
2. **Mutable user preferences (`$XDG_CONFIG_HOME/kirios/`):** widget layout, wallpaper choice, chosen power mode, non-sensitive app preferences. Change immediately; optionally export to Git snapshots.
3. **Sensitive or high-churn state:** SSH keys, cloud credentials, browser sessions, application databases, user files. Keep outside public Git and Nix store. Use age/SOPS or password manager where appropriate; separate backup process for large/high-churn data.

## Why integrated Home Manager

A system rebuild should bring both the correct system and declared user programs. Integrate Home Manager into the NixOS module graph. Match release branches and pin in flake. The caveat: this couples home package changes to privileged rebuild; `kirictl` can offer an advanced standalone-home mode later, but **v1 has only one authoritative activation path**.

**Implementation requirement:** Read [`18-personal-profile-integration.md`](18-personal-profile-integration.md). Cloning a repo does not by itself make it part of a pinned flake; v1 uses a schema-validated profile adapter and generated module overlay, not an unrestricted remote `home.nix` import.

## Sync flow: restore on first boot

1. User picks `Skip`, `Connect Git`, or `Import local folder`.
2. Auth via Git credential manager/browser SSH agent; never embed tokens in command strings/logs.
3. Fetch repository into user-owned state directory with branch/revision displayed; verify ownership/trust and optional signed commit policy.
4. Validate schema and detect conflicting local managed files before activation.
5. `kirictl sync plan`: dry-run home/package changes and file conflicts; show backup/stash options.
6. Snapshot conflicting user files to dated private backup, then apply only after consent.
7. If encrypted secrets requested: unlock and deploy with restricted permissions; no leak to `/nix/store` or `systemd` unit command arguments.
8. Activate Home Manager through the current flake with an authorized system switch. Keep current generation selectable.
9. Save non-secret sync metadata; failure leaves a clear recovery path.

## Sync flow: ongoing

- **Pull:** fetch changes, display diff, detect local modifications, choose merge/keep-local/keep-remote (never blanket overwrite).
- **Push:** export *selected* mutable settings to separate profile path; scan for secrets before staging and require explicit push confirmation.
- **Apply:** validate flake + managed file impact, run rebuild, verify session.
- **Status:** current Git SHA, pending changes, last successful sync, decrypted secret provider status (not values).
- Offline mode retains local state and queues nothing dangerous or ambiguous automatically.

## Recommended repository structure

```text
my-profile/
  home.nix
  profiles/{common,xps,thinkstation,arm-vm}.nix
  preferences/{desktop.json,widget-layout.json}
  secrets/{README.md,personal.yaml.age}       # encrypted ciphertext only
  README.md
```

Use a **personal profile adapter** that reads a restricted validated manifest from a Git checkout and generates approved declarative options without executing arbitrary unreviewed `activation` scripts. Pure Nix evaluation can still produce powerful system changes; show applied changes and treat repository trust seriously.

## Machine differences

- Common user shell and development preferences inherited everywhere.
- Host overlays for battery widget, monitor configuration, hardware launcher, optional x86_64 gaming.
- Host identity selected by explicit hostname/profile, not Git repository location.
- Keep runtime overrides separate from Home Manager-managed paths.

## Encryption design

- SOPS-nix (system and deployed secrets) or age for personal encrypted files. Final option depends on credential/unlock UX and privilege ownership.
- Allow account-free local usage; Git sync and secret restore are optional.
- Never put decrypted material into environment variables exported globally if avoidable; prefer file descriptors/restricted-permission files.
- Provide recovery documentation for lost keys and explain that encryption cannot be bypassed.

## Ownership migration policy

If an existing `~/.config/hypr` or shell file conflicts with a Home Manager symlink, preview conflict and back up original. Never force `home-manager` backup extension and overwrite in a hidden, irreversible manner.

## Acceptance tests

- Fresh user restores expected shell and theme.
- Skip onboarding gives usable desktop.
- Two hosts share common profile but keep correct monitor/device settings.
- Sync conflict preserves original bytes and reports a precise path.
- Wrong age key fails without leaking ciphertext/plaintext to logs.
- Rebuild preserves mutable widget settings.

References: https://nix-community.github.io/home-manager/nix-flakes.html ; https://github.com/Mic92/sops-nix ; https://wiki.nixos.org/wiki/Flakes/en
