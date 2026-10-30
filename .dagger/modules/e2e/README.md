# Integration checks

The root `dagger.toml` registers this module. Run its check by name:

```sh
dagger check --generated=false e-2-e/legacy-caller-check
```

The root `README.md` lists every check and the validated CLI and engine pair.

## Coverage

| Check | Contract | Cap | Git download |
| --- | --- | --- | --- |
| `e-2-e/legacy-caller-check` | A module that declares `v0.21.9` calls this checkout. The helper keeps its own `v1.0.0` API selection. | 180 s | None |

The cap is a proposed acceptance limit, not a measured runtime.

## Telemetry

`fixtures/legacy-caller` calls the helper directly. The calls stay in the trace
of the `dagger check` command, with no downloaded CLI and no Go build.

Read a result with this command:

```sh
dagger trace <trace-id> --check e-2-e/legacy-caller-check
```

## Go SDK coverage

This module no longer runs the Go SDK. The root `README.md` explains the removal
and the lost per-PR Go compilation against the candidate helper.
