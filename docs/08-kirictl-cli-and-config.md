# 08 · `kirictl` CLI and configuration contract

`kirictl` is the canonical human/automation control plane for KiriOS. **Do not use `homectl`**: systemd-homed already provides that executable for managing home directories. The UI and CLI should share typed APIs, not duplicate Nix shell snippets.

## Command tree (project proposal)

```text
kirictl status [--json]
kirictl doctor [--json] [--fix --plan-only]
kirictl hardware probe [--json]
kirictl profile list | show NAME | enable NAME | disable NAME
kirictl packages list | add NAME [--scope system|home] | remove NAME | plan
kirictl theme list | set NAME | preview NAME
kirictl widgets list | enable NAME | disable NAME | layout export|import FILE
kirictl performance status|capabilities|mode MODE|auto on|off [--json]
kirictl gaming status|diagnose|session list
kirictl sync status|init URL|plan|pull|push|apply|export|restore
kirictl apply [--dry-run] [--host NAME]
kirictl generations list | diff A B | rollback [--to GEN]
kirictl update check|plan|apply
kirictl logs [--component installer|performance|sync|desktop]
```

## Operating model

- Commands modifying **live user preferences** can write XDG files without root.
- Commands changing **NixOS state** modify a user-owned *declarative intent manifest* or stage a diff; an authorized privileged service validates and applies the resulting derivation.
- `kirictl apply --dry-run` evaluates and shows flake/source/inputs, package/module changes, services affected and intended generation. Apply requires authorization; no implicit `sudo` from a widget.
- All mutating commands support `--json` machine-readable status where sensible and `--dry-run` unless they only change ephemeral live state.
- Failures leave manifests and generations consistent; use atomic file replace on XDG preferences.
- Avoid writing arbitrary Nix code from unsanitized YAML inputs. Schema keys map to allowlisted Nix modules/packages. Any free-form Nix config editing is an explicit advanced workflow with file review.

## Exit-code contract

| Code | Meaning |
|---:|---|
| 0 | Success |
| 2 | Invalid arguments / schema |
| 3 | Unsupported hardware or feature |
| 4 | Authorization required or denied |
| 5 | Conflict requiring review |
| 6 | Network/cache dependency unavailable |
| 7 | Evaluation/build/activation failed |
| 8 | Unsafe operation rejected |
| 9 | Backend unavailable |

## JSON envelope

```json
{
  "schemaVersion": 1,
  "ok": true,
  "requestId": "req-123",
  "data": { "currentMode": "balanced" },
  "warnings": []
}
```

On error include `{ "ok": false, "error": { "code": "E_UNSUPPORTED", "message": "...", "recoverable": true } }` and omit secret fields.

## First implementation choices

- Language: Rust preferred for typed CLI/service with fast startup; Go acceptable with test coverage. Quickshell/QML for UI.
- Configuration: human-editable TOML or YAML + strictly versioned JSON Schema; prefer TOML for stable round-trip editing or a preserving parser for YAML comments. `contracts/profile.example.yaml` is illustrative.
- Use a `kirictl` client library shared by Welcome, Performance Center and diagnostics.
- Never rewrite `flake.nix` via regex. Render typed configuration into a controlled generated `.nix` overlay or persisted manifest consumed by Nix modules.

## Commands: behavior detail

### `kirictl profile enable gaming`

Validate arch x86_64, runtime graphics capability and user intent; append `gaming` to manifest, show `kirictl apply` diff; do not open Steam automatically or silently switch session.

### `kirictl performance mode performance`

Request temporary change via performance service; show supported/unsupported reasons and active inhibitors. Never claim GPU OC or change kernel. Preserve prior effective state and provide `balanced` reset.

### `kirictl sync pull`

Fetch and show diff. If conflicts exist return exit 5 and require user choice. No automatic `git reset --hard` or `git clean -fd`.

### `kirictl rollback`

Use NixOS generation tooling with clear distinction between changing active generation and selecting a boot generation. Warn that it does not undo user data, external databases, firmware changes or all side effects.

## Abuse tests

Untrusted profile file attempts shell injection, package name injection, symlink/path traversal, concurrent apply and unprivileged access to privileged methods. All should be blocked with structured failures.
