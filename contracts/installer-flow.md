# Installer screen-state contract (proposal)

## State graph

`WELCOME -> NETWORK -> PROBE -> TARGET -> LAYOUT -> ACCOUNT -> FEATURES -> PLAN -> CONFIRM -> EXECUTE -> FINISH`

Every pre-confirm state supports BACK. `EXECUTE` only starts from `CONFIRM` with exact matching disk ID and a freshly validated InstallPlan. No destructively mutating callbacks in `WELCOME..PLAN`.

## Transition guards

- `NETWORK -> PROBE`: release metadata and cache available OR user chooses diagnostic-only mode.
- `TARGET -> LAYOUT`: at least one verified safe disk, disk selected explicitly.
- `PLAN -> CONFIRM`: schema valid, release trusted, `disko` plan inspectable, no mounted installer disk.
- `CONFIRM -> EXECUTE`: same by-id identity, serial/model/size and device major:minor; typed erase phrase; executor re-checks target identity.

## Separate concerns

UI owns presentation and redaction; core validator owns schema and cross-arch validation; privileged executor owns Disko/NixOS install; installation journal owns resumability of safe stages. The ISO must retain a shell and network diagnostics even when UI is broken.
