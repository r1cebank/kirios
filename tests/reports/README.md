# Validation Reports

Store small, sanitized evidence reports here. A report records an observation, not an intention. Planned commands belong in [the validation guide](../../docs/implementation/validation.md).

Use `YYYY-MM-DD-milestone-or-topic.md` filenames. Include:

- Milestone/ticket and status: passed, failed, blocked or not run.
- Git revision, lockfile digest/input revisions, build and test host architecture/OS, tool/firmware versions.
- Exact commands, exit codes and observations; distinguish evaluation, build, ISO boot, install, reboot and desktop results.
- Artifact paths/hashes and sanitized log/screenshot locations. Keep large artifacts outside Git.
- Negative/safety tests, real versus mock providers and security implications.
- Failures, unrun checks, hardware unknowns and next ticket.

Never commit credentials, passwords/hashes, keys, personal diagnostic dumps, VM disks, ISO files or full unreviewed logs. Reports must describe the limits of rollback and data recovery where relevant.

## Report Index

| Report | Scope |
|---|---|
| [2026-11-07 planning](2026-11-07-planning.md) | Specification review, environment observations and documentation validation; no implementation gate passed |
