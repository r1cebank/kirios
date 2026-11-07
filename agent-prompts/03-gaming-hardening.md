# Prompt 03 · Steam/Proton, developer compatibility and release hardening

Read `AGENTS.md`, `docs/05-hardware-and-platforms.md`, `docs/11-gaming-stack.md`, `docs/12-developer-experience.md`, `docs/13-updates-and-security.md`, `docs/14-test-plan.md`, and M6-M8 roadmap gates. Assume earlier milestones passed.

Implement architecture-gated Steam/Proton/Vulkan 32-bit graphics, GameMode, MangoHud and optional Gamescope greeter session on x86_64. Begin with desktop Steam; only enable standalone Gamescope session on validated GPU/driver. Create diagnostic workflows for VS Code launched from GUI, system Node, NVM-installed Node, MCP stdio child spawn, `nix-ld`, rust/cargo/python and Podman/Distrobox. Confirm RTX 6000 identity and XPS dGPU before making proprietary graphics choices. Keep ARM64 guest in console/desktop fallback without unsupported gaming modules.

Add update/signing/recovery workflows and performance A/B comparisons under controlled conditions. Deliver evidence tables with real game, GPU and driver version; do not claim every anti-cheat game works and do not claim optimized kernels are faster without measurements.
