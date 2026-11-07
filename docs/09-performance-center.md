# 09 · Quickshell Performance Center specification

## Goal

A fast, accessible Quickshell widget that communicates **measured workload state** and safely switches power policy. It must never become a root shell script launcher. Should consume low resources while idle and degrade gracefully on ARM VM/unrecognized sensors.

## Layouts

### Compact top-bar button

`[ ◉ Balanced | CPU 9% | GPU 3% | Battery 72% ]` with automatic hide of unavailable metrics. Clicking opens flyout.

### Flyout

```text
Performance Center                           [⚙]
Mode    [ Eco ] [ Balanced ● ] [ Performance ] [ Auto ]
Reason  Balanced (on AC, no GameMode clients)
CPU     14%   3.1 GHz   58°C     [sparkline]
GPU      2%   9 W       42°C     [sparkline] (optional)
RAM    18.2 / 64 GiB              [usage]
Battery 78%   discharging  (optional)
Active optimizers: GameMode inactive · power policy balanced
[See workload details]  [Open diagnostics]
```

### Full panel

Hardware capability badges (neutral UI text), mode explanations, per-process top-N snapshot (permission-filtered), power/thermal graphs, active games, profile automation settings, history export, troubleshooting.

## Data architecture

Quickshell <-> unprivileged `PerformanceService.qml` <-> local read-only API/D-Bus `org.kirios.Performance1` <-> performance daemon <-> kernel/UPower/hwmon/GPU telemetry, supported power backend.

Start V1 with `kirictl performance status --json` polling <=1 Hz; move to signal-driven D-Bus after contracts and daemon are stable. Avoid launching a fresh CLI every 100 ms. Default graph sampling 1 Hz when panel is visible, 0.2 Hz when hidden or minimized, paused on session lock. Redraw only changed properties.

## Capabilities and error states

- If battery unavailable: omit battery data entirely.
- If GPU APIs unavailable or permissions insufficient: display `GPU telemetry unavailable`, not zero usage.
- Temperature values must include sensor source and units, nullable when unavailable.
- Expose whether performance mode is actually supported; powerprofiles daemon may offer only eco/balanced.
- Show “GameMode active for 1 client” only from the authoritative backend, not inferred from window title.
- If backend disconnects: last known values visibly stale; mode selectors disabled; retry with backoff.

## UI events

`onModeRequested(mode)` -> frontend validates enum -> backend checks capabilities/authorization -> request ack -> UI updates **after successful effective state**; on failure return reason and remain previous mode.

## API shape

See `contracts/performance-status.example.json` and `contracts/performance-dbus.md`. Include `requestedMode`, `effectiveMode`, `reason`, `capabilities`, optional sensors, and last update timestamp. Distinguish *policy request* from *actual effective profile*.

## Design and accessibility

- Neutral system aesthetic with theme tokens and minimum readable font sizes.
- Keyboard navigation, screen-reader descriptions where available, high contrast/reduced motion, DPI scaling.
- Unit conversion for °C/°F and GiB/GB only in presentation layer.
- Avoid gradient-heavy/redrawn canvases; short trend windows retained in memory, capped buffer.

## Acceptance

1. Changing Eco->Balanced immediately updates backend and UI on XPS, without relaunching session.
2. Unsupported Performance on VM results in precise disabled state, not crash.
3. Active GameMode temporarily overrides auto balanced; user override precedence is documented and honored.
4. Missing NVIDIA telemetry doesn't break CPU/RAM widget.
5. Background CPU cost and redraw frequency measured; no high-frequency polling loops.
