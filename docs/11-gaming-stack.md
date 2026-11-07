# 11 · PC gaming: Steam, Proton and SteamOS-style session

## Scope

Gaming targets are **x86_64 physical systems**, with hardware-dependent support. Standard Steam + Proton runs through Steam's compatibility tooling; distro does not install a separate Windows filesystem or package each game with Nix.

## Two sessions

- **Desktop mode:** Hyprland + Quickshell + desktop Steam, standard productivity/dev use, normal compositor.
- **Console mode (opt-in):** greeter-selectable Gamescope + Steam Big Picture/gamescope session, gamepad-friendly interaction. Gamescope session availability must be validated per GPU/driver; NVIDIA support is more variable than a Steam Deck-like AMD environment.

The default is Desktop. Game session errors return safely to greeter rather than leaving user with a black screen or auto-login loop.

## Base NixOS settings (illustrative, verify pinned options)

```nix
{ pkgs, lib, config, ... }: {
  hardware.graphics.enable = true;
  hardware.graphics.enable32Bit = true;  # required for many x86_64 Steam/Proton paths
  programs.steam.enable = true;
  programs.gamescope.enable = true;
  programs.gamemode.enable = true;
  environment.systemPackages = with pkgs; [ mangohud protonup-qt ];
  # Enable gamescope session in a separately tested host feature:
  # programs.steam.gamescopeSession.enable = true;
}
```

Gate this entire module to `x86_64-linux` and a supported graphics profile. GPU Vulkan/32-bit driver packages require matching hardware support. Keep unfree package consent and licensing policies explicit.

## Component inventory

| Core | Optional |
|---|---|
| Steam, Proton (Steam-managed), Vulkan, correct 32-bit graphics, GameMode | GE-Proton, ProtonUp-Qt, Gamescope session, MangoHud overlay, Lutris, Heroic |

Do not ship unverified community Proton binaries silently; optional download/selection remains user-controlled. Steam game compatibility is game-specific, especially kernel anti-cheat, launchers, codecs and multiplayer.

## Useful command checks

```text
steam                         # GUI launch in desktop
vulkaninfo --summary           # host Vulkan smoke test, package availability verified
mangohud gamemoderun %command% # Steam launch-options form, only if installed
```

`%command%` is a **Steam launch-option placeholder**, not valid as a normal shell command. For Gamescope, prefer tested session integration instead of manually constructing fragile flags for all GPUs.

## Hardware-specific details

- **XPS:** verify whether it has only Intel iGPU or an NVIDIA/AMD discrete GPU. If hybrid, test PRIME/offload and battery drain. Steam launch on unsupported iGPU workloads may be limited.
- **ThinkStation:** identify RTX 6000 generation before choosing Nvidia open/proprietary kernel module and driver. Validate Wayland modesetting and 32-bit Vulkan; Gamescope standalone session requires explicit test. CUDA should be a separate developer feature.
- **ARM64 VM:** no default Steam/Proton feature. CPU translation and guest GPU virtual support are separate research tracks, not release gates.

## Features and defaults

- No automatic GPU overclock/power limit edits.
- GameMode integration managed per game; user-selectable performance profile fallback.
- MangoHud opt-in per game; overlay shouldn't appear on screen-sharing or sensitive apps by accident.
- Explicit session UX: `Switch to Gaming Mode` should save user session safely, confirm session end, and start greeter-selected console session.
- No Steam autologin or credentials captured in ISO.

## Test coverage

- Steam starts from desktop entry with GUI session PATH and Vulkan access.
- Download a *known compatible* test game using user's account/permission, enable Proton where needed; launch and exit.
- Verify `gamemoderun` activates/deactivates, MangoHud can attach where supported.
- Compare frame times native Hyprland vs Gamescope, documenting both.
- Session switching -> return to greeter -> desktop data remains intact.

References: https://wiki.nixos.org/wiki/Steam ; https://github.com/Jovian-Experiments/Jovian-NixOS
