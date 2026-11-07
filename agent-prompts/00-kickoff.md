# KiriOS | Prompt 00 · Kick off the repo and installer

You are the lead systems engineer building a custom, high-performance and efficient NixOS-based workstation distribution. The attached ZIP is the source architecture and acceptance spec. Read `README.md`, `AGENTS.md`, `docs/00-product-and-decisions.md`, `docs/01-architecture.md`, `docs/02-flake-and-repository.md`, `docs/03-iso-and-installer.md`, `docs/04-disk-and-recovery.md`, `docs/14-test-plan.md` and `docs/15-delivery-roadmap.md` before editing.

Implement **M0 then M1 only** in the working repository. Produce a pinned NixOS 26.05 flake, shared modules and host stubs for `xps`, `thinkstation`, `arm-vm`, `vm-test`, an x86_64 custom network installer ISO, a minimal text installer with separate plan/execute steps, safe Disko install into a disposable VM disk, optional recovery shell, and an end-to-end boot/install/reboot test. Build ARM64 ISO if a native ARM64 Linux runner is available; otherwise provide correct outputs and document inability to test.

Security is non-negotiable: no destructive action without explicit disk identity confirmation; no unverified remote shell scripts; don't hardcode secrets, private usernames, tokens or device paths. Use a VM target first. Prefer real end-to-end boot evidence over fake scaffolding. If upstream option names differ, inspect locked versions and create an ADR instead of fabricating an implementation.

Deliver after each milestone: changed files, design decisions, exact test commands/results, ISO artifact hash/path, failures, next task. Stop before working on the full desktop, sync, performance widget or gaming stack until the installer gate passes.

## Canonical identity

The project is **KiriOS**, not an unnamed custom NixOS project. Follow `docs/19-brand-and-naming.md`. Use `kirictl` for the public CLI, `kirios` for machine-readable IDs and file paths, and `KiriOS` for the public distribution label. Do not introduce other project CLI names or reuse systemd’s `homectl`.
