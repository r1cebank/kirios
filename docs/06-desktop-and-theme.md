# 06 · Hyprland, Quickshell, theming and welcome UX

## Desktop contract

First boot has a greeter, UWSM-managed Hyprland session, readable wallpaper/panel, network UI, app launcher, terminal, browser, file manager, notifications and settings. Minimal usable UI must not require Git login. Use the NixOS `programs.hyprland.withUWSM` path and set Home Manager's `wayland.windowManager.hyprland.systemd.enable = false` to avoid duplicate session management.

## Quickshell architecture

```text
shell.qml
  components/{Button,Card,Icon,Slider,Tooltip,Overlay}.qml
  panels/{TopBar,Dock,SideBar}.qml
  widgets/{Clock,Media,Network,Bluetooth,PerformanceCenter,Tray}.qml
  overlays/{Launcher,QuickSettings,Power,NotificationCenter}.qml
  services/{PerformanceService,SettingsStore,ThemeService}.qml
  themes/{defaults.json,tokens.qml}
```

Quickshell should load per-user preferences, render safe defaults if files are missing/corrupt, and update relevant widgets without `nixos-rebuild`. Avoid executing arbitrary shell fragments from UI configuration. Systemd user service should be tied to graphical session; don't duplicate startup through Hyprland `exec-once` and systemd.

## Theme system

- Canonical palette/tokens: accent, contrast/surface, text, borders, radii, spacing, font, animation budget.
- Derived targets: Quickshell/QML, Hyprland decorations, GTK/Qt where compatible, terminal, lock screen, greeter assets.
- Use Stylix where it works well, but allow local shell-only changes without triggering a Nix rebuild.
- Personal style variants (for instance teal-accent) are not user identity; shared ISO defaults are generic and replaceable.
- Accessibility: WCAG-like contrast testing, reduced motion, keyboard navigation, touchpad support, DPI scaling, high-refresh display support.

## Live preference ownership

Store mutable state in `$XDG_CONFIG_HOME/kirios/preferences.json` and layout files. Home Manager should install defaults in read-only `/etc/xdg/kirios/defaults.json` or an immutable reference and copy them on first run **only if absent**. Never symlink a user-editable target to a read-only Nix store path. `kirictl sync export` includes an explicit snapshot of chosen preferences.

## First boot wizard

Stages: network test -> choose theme -> displays/scaling -> optional Git repo/auth -> preview home sync -> optional secrets unlock -> choose developer/gaming packages -> `kirictl doctor` -> finish. Every stage is skippable except essential local account setup. Resume stage state in XDG state, not Nix store.

## Fallbacks

- If Quickshell crashes repeatedly, fallback to a minimal Hyprland config with terminal and launcher.
- If Hyprland cannot start (VM unsupported graphics), choose a supported simpler graphical or TTY login; no forced auto-login loop.
- External wallpapers/fonts/assets are packaged with proper licensing, avoiding runtime download for first boot.

## UX acceptance

User can alter wallpaper, accent, widget visibility and positions live, quit/restart Quickshell, reboot, and see the same choices; a NixOS rebuild doesn't silently erase them.
