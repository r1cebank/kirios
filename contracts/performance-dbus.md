# Performance service IPC contract (proposal v1)

This is a **design contract**, not an installed service. Prefer reusing existing `power-profiles-daemon` D-Bus API for direct controls where practical. Add a custom aggregation service only if telemetry/policy orchestration requires it.

## Identity

- Bus name: `org.kirios.Performance1` (provisional, rename with branding)
- Object path: `/org/kirios/Performance1`
- Interface: `org.kirios.Performance1`

## Methods

- `GetStatus() -> (s json)`: current validated `performance-status` document.
- `GetCapabilities() -> (s json)`: host-supported modes/readable sensors.
- `RequestMode(s requestedMode, u timeoutSeconds) -> (s json)`: request an enum mode; backend replies with accepted/effective status, never raw sysfs access.
- `SetAutoPolicy(s json) -> (s json)`: schema-validated policy update; authenticated as necessary.

## Signals

- `StatusChanged(s json)`: emit at bounded cadence and when effective mode changes; don't emit high-frequency raw per-process telemetry.
- `CapabilitiesChanged(s json)`: hotplug/driver change.
- `ErrorOccurred(s code, s message)`: redacted, user-actionable.

## Policy rules

- Valid modes: `auto`, `eco`, `balanced`, `performance`. `gaming` is a contextual GameMode event/policy, **not** automatically a hardware-supported CPU profile.
- Reject unsupported mode with explicit typed error; never report desired mode as actual mode after backend rejection.
- Read-only methods require no privileged user secrets; mutating methods use Polkit and a narrow allowlist.
- All request logs omit credentials and personally identifiable process arguments.
- The backend should not expose arbitrary command execution or raw file write RPCs.

## Observability

Every status contains timestamp, source (`real|mock`), capabilities, policy, nullable metrics, warnings. Graph buffer and UI polling rate are client concerns; service must avoid unnecessary high-frequency polling when no client needs it.
