# 14 · Automated test plan and release gates

## Test pyramid

1. **Schema and unit:** profile JSON Schema, CLI parse/exit codes, settings preference migrations, policy state-machine transitions, correct generated Nix fragments.
2. **Nix evaluation:** `nix flake check`, host output evaluation on x86_64 and aarch64, option assertions, cross-architecture package gating, unsupported configurations rejected.
3. **ISO build:** both architectures build on native runners, UEFI boot, network setup and installer UI.
4. **VM E2E:** a disposable encrypted and unencrypted virtual disk can be installed; post-reboot login desktop or safe fallback; offline reboot; repeated runs.
5. **Physical hardware smoke:** separately on XPS and ThinkStation after VM milestone acceptance and complete backup.
6. **Gaming/efficiency:** hardware-specific Vulkan/Steam/Proton and power/thermal metrics.

## Matrix

| Test | x86 VM | ARM VM | XPS | ThinkStation |
|---|---:|---:|---:|---:|
| Flake eval/ISO | Must | Must | config only | config only |
| UEFI install/reboot | Must | Must | Later | Later |
| LUKS Btrfs install | Must | Recommended | Later | Later |
| Hyprland UI | Must if VM GPU supports; fallback otherwise | best effort | Must | Must |
| Home Manager sync | Must | Must | Must | Must |
| `kirictl doctor` | Must | Must | Must | Must |
| Performance modes | Mock backend | Unknown/unsupported | Must | Must |
| Node/VS Code/MCP | Must where GUI supported | Terminal at minimum | Must | Must |
| Steam/Proton/Vulkan | x86 GPU test if possible | N/A | Must if supported GPU | Must |
| Gamescope session | Conditional | N/A | Optional | Conditional |

## Installer destructive safety tests

- Reject blank/unavailable/mounted/external installer disk.
- Reject mismatched serial/size during confirmation or before execution.
- Reject incorrect typed erase phrase.
- Network unavailable before partition -> zero disk writes.
- Crash after format -> no unsafe automatic format rerun.
- Reject unsigned/invalid release manifest.
- No credentials appear in logs/install report.

## Sync tests

- Nonempty home conflict is preserved and reported; no silent overwrite.
- Offline/invalid Git remote falls back to usable desktop.
- Encrypted secrets wrong key -> failed stage but no leak and no unusable desktop.
- Cross-host configuration doesn't import GPU-specific binary on ARM.

## Power tests

- Mode request -> effectiveMode reports actual backend result, not just desired state.
- GameMode enters/exits and restores previous profile after game crash.
- Backend restart preserves safe default; thermal/unsupported condition overrides requests.
- Missing battery/GPU sensors return null/unsupported, not false numeric zeros.
- Idle widget polling <= project budget; freeze background graphs when hidden.

## Developer tests

- GUI-launched VS Code can run absolute-path Node MCP smoke.
- NVM-installed Node runs with compatibility settings; external tools fail visibly with troubleshooting.
- Python venv, Cargo, Podman sample workflows.

## Gaming tests

- `vulkaninfo --summary` reports correct device; check 32-bit graphics path.
- Steam login works through GUI, Proton game launches, exits and input/audio works.
- MangoHud and GameMode optional test; Gamescope session separate and driver-gated.
- Anti-cheat incompatibility is documented, not a release blocker for specific protected games.

## Performance methodology

Record baseline versus candidate kernel/settings with repeated measurements, same GPU driver, power source, graphics settings, thermal conditions and pinned revision. Include not just average FPS but 1% lows, frametime variance, idle watts, CPU/GPU temperatures, build times and stability. No promotional claims without measurements.

## Final release checklist

- ISO boots and installs a new VM; signed checksums valid.
- Backup/recovery and previous generation boot tested.
- First desktop login works without Git sync.
- CLI and Performance Center backend tested with error paths.
- Zero passwords/secrets in artifacts/logs/Nix store sample inspection.
- Known unsupported hardware/features included in release notes.
