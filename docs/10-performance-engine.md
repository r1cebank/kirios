# 10 · Performance policy engine and benchmark methodology

## Philosophy

Use **performance on demand**, efficient idle and measured tradeoffs. A tuned kernel/optimized ISA does not automatically improve battery life, game frametimes or GUI latency. Start with stock kernel and standard cache binaries. Benchmark alternatives behind flags.

## State machine

Inputs: `userMode = eco|balanced|performance|auto`; AC/battery, hardware supported profiles, GameMode clients, thermal conditions, explicit user time-limited overrides. Effective policy is derived, never inferred from a single instantaneous CPU spike.

Priority (highest first): hardware/thermal safety -> admin policy -> explicit time-limited user override -> active GameMode request (within allowed policies) -> auto policy -> balanced fallback. Record `reason` and expiration. User choosing Eco should not be silently promoted to Performance by a game unless explicit “allow game overrides” preference is on.

## Examples

- `auto`, battery, no active game -> eco when supported; default may remain balanced depending thermal/battery policy.
- `auto`, AC, desktop idle -> balanced.
- `auto`, AC, GameMode active -> performance **only if supported and user permits**; revert when GameMode client closes, including crash.
- manual performance -> stay until user changes or timeouts (default configurable), but thermal safety always wins.
- ARM VM reports `unsupported` for physical CPU governor/GPU controls; telemetry stays functional.

## Backend integrations

- `power-profiles-daemon` on appropriate laptop hardware; choose it **or** TLP/auto-cpufreq, never simultaneous conflicting writers.
- GameMode for game-scoped tuning; backend may observe its status but avoid independently duplicating every GameMode tweak.
- UPower/hwmon for laptop battery/thermal data; `procfs`/sysfs for CPU/memory; GPU provider abstraction for NVML/NVIDIA/AMD/Intel where available.
- CPU pinning, sched-ext, frequency caps, GPU power limits and overclocking are **out of scope** until benchmarks and access controls justify them.

## Polkit and privilege design

- Read-only telemetry runs without root if kernel permissions allow.
- Narrow system service offers `SetPowerMode(enum)` and `GetStatus()` only; uses Polkit for any privileged action. Do not allow arbitrary sysfs path writes or command strings from QML.
- Restore prior mode on service restart/crash based on persisted *desired* policy plus capability detection; clean up temporary GameMode state on exit.
- Consider an established power service instead of custom root daemon for v1; new service must justify why upstream API insufficient.

## Benchmark plan

Per-host A/B runs at same firmware, temperature range, power source, package versions and game settings. Record ambient conditions, NixOS generation, kernel, driver, GPU power profile, PSU and display refresh, number of repetitions, median + spread.

| Target | Baseline metrics | More important than raw peak FPS |
|---|---|---|
| XPS | idle draw, battery runtime, 1% low FPS, boot, build time, thermal throttling | sustained responsiveness & acoustics |
| ThinkStation | 1% lows, frame time, CPU build throughput, CUDA/GPU performance, power | stability, repeatability, graphics correctness |
| ARM VM | login latency, RAM, load, installation time | correctness and minimal overhead |

Use `hyperfine` for CLI timings, `systemd-analyze` for boot, MangoHud for game frametimes, `powerstat`/UPower or hardware meter where valid. Do not compare graphics workloads across heterogeneous machines as kernel benchmarks.

## Performance release gate

A tuning change ships only with a documented repeatable improvement, or clear rationale such as improved power/fan behavior, with no stability regression. Keep old profile selectable. Avoid global x86-v4 rebuilds that destroy binary-cache coverage.
