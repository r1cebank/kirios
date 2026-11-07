# 01 · System architecture and contracts

```text
Versioned source/flake.lock             Personal Git profile (optional)
    |                                             |
    +--> ISO builds (x86_64, aarch64)             +--> first-boot sync
    |       |                                             |
    |       +--> network / installer TUI                  +--> Home Manager managed files
    |       +--> disko (guarded)                           +--> mutable XDG preferences
    |       +--> nixos-install                             +--> age/SOPS secret unlock
    |                                                     |
    +--> NixOS configurations ---> core/desktop/dev/gaming/performance
                           |                  |           |
                           |                kirictl CLI <--+
                           |                  |
                           |             local system service (privileged narrow API)
                           |                  |
                           +--> UWSM/Hyprland + Quickshell Performance Center
                           +--> Steam + Proton (+ optional Gamescope session)
```

## Configuration ownership table

| Asset | Source of truth | Apply mechanism | Root needed? |
|---|---|---|---|
| Kernel/boot/filesystem/system services | Flake NixOS modules | `nixos-rebuild` | Yes |
| System packages/drivers/Steam | Flake system modules | `nixos-rebuild` | Yes |
| Home programs, managed dotfiles | Home Manager module | Integrated `nixos-rebuild` | Via system activation |
| Wallpaper/widget layout/accent chosen live | Per-user XDG file | Quickshell reload | No |
| Per-user app session state | XDG state | App-native | No |
| Per-machine mutable calibration/preferences | XDG + host-specific config | App/backend | Sometimes |
| Passwords/tokens | SOPS/age or password manager | Unlock/runtime deployment | No plaintext store strings |
| Performance current mode | Backend ephemeral state | Polkit-gated action if needed | Policy-dependent |
| Nix flake revision applied | System closure + metadata | `kirictl status` | Read-only |

## Boundaries

- **Installer frontend:** TUI asks questions, makes no changes on its own. Passes a validated typed plan to executor.
- **Installer executor:** privileged isolated process with allowlisted commands; writes an install journal; never accepts raw shell fragments.
- **Flake generator:** deterministic pure inputs + typed profile config -> a valid host configuration; no guessing of a real disk at evaluation time.
- **kirictl:** human/JSON CLI, manages manifests and reports diffs; route privileged operations via NixOS/Polkit service or sudo authorization, no hidden root escalation.
- **Performance daemon:** reads safe telemetry, exposes state and controlled actuators; Quickshell remains unprivileged UI.
- **Sync manager:** shallow clone/fetch pinned user-owned Git, merge/preview/conflict management, secret decrypt; never silently overwrite a non-empty home directory.

## Shared buses and interfaces

1. `kirictl --json ...` is the initial stable automation interface.
2. Performance backend should eventually expose a **versioned local D-Bus API**. V1 can use read-only JSON bridge (`kirictl performance status --json`) to avoid early daemon complexity.
3. Quickshell theme engine consumes JSON/TOML from an XDG path and updates immediately. System theme defaults can be generated into `/etc/xdg/kirios`.
4. All backend errors are structured: `{ "code": "E_...", "message": "...", "recoverable": true }`.

## Security trust boundaries

- A live ISO is untrusted relative to a user's existing disk; requiring typed consent is mandatory.
- Personal Git repository may contain hostile code; scripts/hooks require review or explicit opt-in during restoration.
- User widgets must not access unrestricted root D-Bus methods; restrict to narrow policy-approved actions.
- Game clients, Proton/Wine apps and browser downloads are untrusted workloads; don't weaken the whole host filesystem for compatibility.

## Offline and recovery behavior

- Live ISO supports Wi-Fi setup, wired network, time synchronization and diagnostics.
- Network install fails **before disk writes** if release metadata, substituter/cache access or required payloads cannot be obtained (preflight downloads where feasible).
- An in-progress interrupted destructive install is **not** safely resumable without rechecking partition/format state; show precise recovery steps. “Resume” only applies to safe post-partition stages with checkpoint verification.
- First boot has a graphical fallback independent of restored user settings.
