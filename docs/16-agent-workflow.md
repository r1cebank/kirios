# 16 · How to drive the coding agent

## Starting prompt

Use `agent-prompts/00-kickoff.md`; give the agent the **whole ZIP** and a new repository, and require it to read `AGENTS.md` and linked specs before coding. The original docs are a baseline; the implementation repository becomes source of truth as tests validate assumptions.

## Work style

1. Read required specs and cite exact document headings in initial plan.
2. Inspect actual toolchain/pinned Nix release and host environment before changing anything.
3. Select **one milestone** and implement end to end, keeping PR/commit scope small.
4. Run non-destructive checks; boot and install only on disposable VM disks.
5. Report evidence: test command, environment, status, logs path and remaining failures.
6. If design conflicts with upstream reality, write `docs/adr/NNN-title.md` explaining change and update affected specs.
7. Do not claim that an ISO works merely because derivation evaluates.

## Interface-first contracts

Before implementing UI, lock `kirictl` error codes/status JSON and performance API. Before implementing installer executor, lock `InstallPlan` fields and confirmation protocol. Before implementing sync, lock ownership and conflict policy.

## Agent prompts

- `agent-prompts/00-kickoff.md`: M0+M1 start.
- `agent-prompts/01-desktop-sync.md`: M2+M3 after installer passes.
- `agent-prompts/02-cli-performance.md`: M4+M5.
- `agent-prompts/03-gaming-hardening.md`: M6+ testing/release polish.

## Evidence format for milestone reports

```text
Milestone: M1
Revision: <git SHA>
Built: <artifact path/hash>
Environment: x86_64 VM, NixOS source revision, UEFI implementation
Tests: exact commands + PASS/FAIL
Safety checks: wrong disk, dry-run, redaction
Demo: install->reboot screen/log screenshots (no secrets)
Known gaps: ARM64 boot, LUKS, GPU
Next ticket: M2-01
```

## Important unknowns

- Dell XPS exact model and GPU.
- ThinkStation Xeon W5 SKU, RTX 6000 exact generation, UEFI Secure Boot preference.
- macOS VM platform: UTM, VMware, Parallels or Apple Virtualization.framework stack.
- Personal Git repository URL and auth mechanism.
- Visual style and logo artwork (distribution name is fixed as **KiriOS**).

Do not block initial VM milestone on these. Use VM fallback defaults and mock providers. Ask only when implementing the respective physical-hardware or personal-sync steps.
