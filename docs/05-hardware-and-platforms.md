# 05 · Hardware matrix and host-specific policy

## XPS laptop

- Known: Intel Core i7-12700H (hybrid P/E architecture), 64 GB RAM.
- Unknown: exact XPS model, iGPU/dGPU presence, Wi-Fi chipset, display resolution/scaling, audio codec, fingerprint reader, SSD topology.
- Defaults: stock compatible kernel, Intel integrated display where practical, `power-profiles-daemon` **or** one alternative power manager, adaptive Balanced default, efficient idle behavior, suspend test, battery monitoring, thermal monitoring. Do not presume x86-64-v4 support.
- Gaming capability: Steam/Proton enabled only once Vulkan and device support pass; if dGPU exists evaluate NVIDIA PRIME/offload and real power draw.
- Test: idle wattage, suspend/resume, USB-C/dock, display refresh, audio, Wi-Fi roam, P/E scheduling under load.

## ThinkStation workstation

- Known: Xeon W5 20-core CPU, 256 GB RAM, RTX 6000 GPU.
- Unknown: exact Xeon SKU (AVX-512 flags), RTX 6000 generation (Quadro/A6000/Ada/etc.), monitor connections, Secure Boot state, disks, system firmware.
- Defaults: stable supported kernel, NVIDIA driver family and open/proprietary kernel-module choice **after GPU ID check**, explicit Vulkan+32-bit graphics support, CUDA as optional developer feature, no aggressive ZRAM tuning, no blind x86-v4 flags.
- Gaming: desktop Steam first; optional Gamescope session tested separately; support HDR/VRR as future host-specific features only after demonstrated driver/compositor support.
- Build server: optional signed Nix binary cache and x86_64 remote builds; avoid exposing builders to untrusted networks.
- Test: CUDA compute, Vulkan 32-bit, game frame times, large compile stability, GPU idle/active power, Nvidia modesetting, recovery boot.

## ARM64 macOS VM

- Known: guest intended as aarch64 Linux VM on Apple Silicon macOS (host laptop M5 from prior discussion).
- Unknown: VM implementation and available virtio GPU/virgl/Vulkan, Linux guest UEFI implementation.
- Defaults: `aarch64-linux`, NixOS stock ARM kernel, no NVIDIA, no x86_64 gaming, no CachyOS x86 kernels, performance controls read-only or no-op when unavailable.
- Graphics tiering: `T0` terminal-only installation; `T1` simple fallback graphical session; `T2` Hyprland when DRM/EGL/Vulkan requirements verified. Never force Hyprland if startup would black-screen the guest.
- Test: ARM ISO boots, installs to virtual disk, networking works, home sync and CLI run, evaluate fallback graphics. Avoid promising Steam/Proton parity.

## Hardware detection API

`kirictl hardware probe --json` should expose observed architecture, CPU vendor/model, physical/VM flag, PCI IDs, graphics render devices, available power profiles, GPU driver, battery paths and whether Steam gaming is supported. Distinguish `supported`, `unsupported`, and `unknown`; unknown disables risky controls.

## Host modules: gates

- Architecture check on **every** native binary and module.
- Proprietary GPU module imported only on matching host/PCI ID or explicit validated override.
- `performance.powerManager` enum with assertion that only one CPU policy service is active.
- A host's disk ID is collected at install time, never inferred from its CPU or hostname.
- Kernel tuning can differ between laptop and workstation, but all custom kernels must retain a fallback boot entry.
