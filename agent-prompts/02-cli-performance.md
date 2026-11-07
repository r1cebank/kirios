# Prompt 02 · `kirictl` CLI and Performance Center

Read `AGENTS.md`, `docs/08-kirictl-cli-and-config.md`, `docs/09-performance-center.md`, `docs/10-performance-engine.md`, `contracts/performance-dbus.md`, and the relevant example JSON files. Assume basic install/desktop/sync work.

Implement M4/M5: typed CLI with JSON envelope and strict error codes, schema-driven declarative profile management, validated dry-run/diff/apply/rollback, read-only diagnostics, safe user preference operations and `kirictl sync`. Then build performance backend with capability discovery, mode state machine, conservative supported power backends, Polkit authorization for privileged controls, and Quickshell compact/flyout Performance Center. Do not shadow systemd's `homectl`. Do not expose unrestricted shell or sysfs writes over a widget/DBus API. Provide mock hardware providers for VM tests, real host probes for future laptop/workstation validation.

Test wrong permissions, missing sensors, unsupported modes, backend crash, gaming override start/exit, actual-vs-requested mode and widget polling budgets. Record exactly what is mock versus physically verified.
