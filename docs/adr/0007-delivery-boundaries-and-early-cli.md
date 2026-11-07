# ADR 0007: Delivery Boundaries and Early CLI Dependencies

Date: 2026-11-07. Status: accepted for implementation planning; not implemented.

## Context

[AGENTS.md](../../AGENTS.md) includes a basic Hyprland/UWSM desktop, Quickshell and smoke test in the initial deliverable. [The roadmap](../15-delivery-roadmap.md) separates the bootable installer into M1 and desktop into M2. The kickoff prompt requests M0/M1 before full desktop work. Calling M1 the MVP would contradict the defined working-desktop outcome.

The README expects `kirictl doctor` in the MVP and the hardware specification defines `kirictl hardware probe`, while the roadmap places the CLI in M4. Sync in M3 also depends on a backend later described in M4. Integrated Home Manager is a foundation input but fuller home configuration is listed in M3. Literal deferral would create duplicate temporary tools or leave earlier milestones without their required interfaces.

## Decision

1. Preserve M0-M8 IDs and ordering. M0/M1 are the first implementation delivery, with installed-system TTY/login as M1's gate. The initial usable desktop MVP is M0-M2 together.
2. Integrate Home Manager in the foundation; provide default managed desktop/user programs in M2. M3 adds personal restoration rather than introducing a competing activation owner.
3. Introduce only the shared Rust validation/probe code and read-only `kirictl hardware probe` needed by M1. Add read-only `status` and `doctor` in M2. Use the specified JSON/error/exit-code conventions from their introduction.
4. Implement the backend subset needed for M3 sync with its owning feature. M4 expands and hardens the same CLI into the full management contract. Early subsets do not complete full M4 tickets.
5. Keep installer executor entrypoints internal. Illustrative `project-install` examples do not establish a second public management CLI.

## Consequences

Every milestone retains a concrete gate. M1 passing is not a desktop claim; M2 must use the real ISO installation path. No broad widget, performance, sync or gaming implementation is pulled into the installer milestone.

Shared interfaces arrive before their first consumer, reducing throwaway implementations. Contract tests begin with those interfaces. Significant later deviations still require their own ADR and updated evidence.

This resolves planning order only. It changes no disk protections, privilege rules, architecture requirements or secret-handling requirements. The current user request authorizes documentation and a stop before implementation.
