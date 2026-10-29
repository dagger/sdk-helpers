# SDK helpers

Use `ModuleManifest` to load, change, and generate Dagger module manifests.
The manifest selects engine API version `v1.0.0`.
Callers can still select `v0.21.9` on that engine. The helper needs the newer
API for its own Workspace operations.

## Generate manifests

Call `manifest.generate(ws, lock: false, legacyJson: true)`.
Both options are optional. These are their defaults.

The method returns a modified `Workspace`. It writes `dagger-module.toml`
at `ws.cwd`. It also writes `dagger.json` when legacy configuration
exists. If `legacyJson` is false, or no legacy configuration exists, it removes
`dagger.json` from the returned workspace. It preserves other files and the
current directory. The input workspace stays unchanged.

TOML output is always enabled. Selection of a future TOML format is not yet
supported.

## Load manifests from a workspace

Call `sdkHelpers.loadModuleManifest(ws)` to load the manifests in `ws.cwd`.
It loads `dagger-module.toml` and `dagger.json` when present. Without either
file, the manifest is empty. Generation keeps the loaded fields that the caller
does not change, such as `include`, `source`, and `codegen`.

The legacy runtime methods keep a loaded module source and engine version
unless the caller passes new values. On an empty manifest, they use the
manifest directory and the running engine version.

## Select the legacy runtime

Each built-in runtime has its own method, such as `withLegacyPythonRuntime`.
Call `manifest.withLegacyRuntime(runtime)` for a runtime that is not built in.
The runtime value is written as `[runtime] source` without changes. It accepts
a built-in runtime name, a path relative to the manifest directory, or a module
reference such as `github.com/dagger/python-sdk/runtime@v1`. An absolute path
is rejected.

A value without `/` and without `.` must name a built-in runtime: `go`, `dang`,
`python`, `typescript`, `php`, `elixir`, or `java`, optionally with `@version`
as the engine accepts it. Write a local path as `./runtime`, not `runtime`. A
runtime loaded from an existing manifest keeps its value as written.

The helper does not resolve a pin for the runtime. With `lock: true`, it keeps
the pin a loaded manifest already had for the same runtime source.

## Add a dependency

Call `manifest.withLegacyRuntimeDependency(module)` with a `ModuleSource`.
The helper uses `module.moduleOriginalName` as the dependency name. A loaded
dependency with the same source keeps its name and settings instead, so an
alias survives generation and no duplicate is added.

Call `manifest.withLegacyRuntimeDependencies(modules)` to make the modules the
complete dependency list. Loaded dependencies that are not in the list are
removed.

For Git dependencies, it writes `module.asString` as the source without changes.
This includes any `@version` or `@commit` present in that value. The engine can
add a version when it loads a source that had no version.

A Git workspace, such as a CI checkout loaded by address, loads its own modules
as Git sources. When the output workspace resolves a Git module's path to the
same source, the module belongs to the workspace, so the helper writes it as a
local dependency instead: a relative path with no pin.

For local dependencies, pass a module from the output workspace. The helper
makes its path relative to `ws.cwd` during generation. Absolute local
paths in loaded manifests are rejected. Loaded relative paths are used as
written, relative to the output manifest file.

## Control pins

A pin is a separate manifest field that stores a selected Git commit.

By default, generated JSON and TOML have no dependency or runtime pin fields.
Legacy JSON blueprint and toolchain pins are also omitted. Sources stay intact,
so a commit in the source still selects that commit.

With `lock: true`, the helper uses the selected commit from each supplied
`ModuleSource`. Local module sources return an empty pin. For dependencies loaded
from JSON or TOML, it preserves existing pins and leaves missing pins absent.
Generation does not resolve sources or look up missing pins.

Loaded pins remain internal to `ModuleManifest`. Generation does not remove
them from that object. The same object can generate output with or without pins.

## Write dang.toml

Use `DangConfig` to load, change, and write a Dang project configuration.
Call `sdkHelpers.loadDangConfig(ws)` to load `dang.toml` in `ws.cwd`. Without
the file, the configuration is empty. Call `sdkHelpers.dangConfig(loadTOML:
file)` to load a `File` instead.

Call `config.withDaggerImport(name, version)` to set
`imports.<name>.dagger`. Each call adds or replaces one named import. Other
settings of a loaded import, such as `service`, are kept.

Call `config.generate(ws)` to write `dang.toml` at `ws.cwd`. It returns a
modified `Workspace` and preserves other files. If the configuration has no
`dang` version, it writes `dang = "2.1"`. Call `config.withVersion(version)` to
write another version. A loaded version is kept.

The output keeps the loaded settings, but not their comments or layout. Keys
are written in alphabetical order.

## Update existing SDK callers

- Replace `withLegacyRuntimeDependency(source, name, pin)` with
  `withLegacyRuntimeDependency(module)`.
- Replace `tomlFile`, `legacyJSONFile`, and `directory` with
  `generate(ws, ...)`. It returns a `Workspace`.
- Use `lock: true` when the SDK must write pins.
- Replace a manifest built from scratch with `loadModuleManifest(ws)` and
  `withLegacyRuntimeDependencies(modules)` to regenerate an existing module
  without losing its fields.

## Run checks

Run each check by name, one at a time:

```sh
dagger check --generated=false manifest/loads-manifest-formats
dagger check --generated=false omits-default-module-source
dagger check --generated=false e-2-e/legacy-caller-check
```

List the check names with `dagger check -l`.

All 26 named checks passed with CLI `v1.0.0-beta.21` (commit `a7a2e64e`) and
engine `v1.0.0-beta.21`.

### Coverage

Each check has an acceptance cap. A cap is a proposed limit, not a measured
runtime. A check over its cap fails acceptance.

| Check | Contract | Cap | Git download |
| --- | --- | --- | --- |
| `manifest/loads-manifest-formats` | JSON only, TOML only, both, and neither. TOML precedence and retained legacy-only JSON fields. | 120 s | None |
| `manifest/writes-dang-entrypoint` | A Dang source file: kind `dang`, source, name, replacement of an earlier entrypoint, and no legacy output. | 120 s | None |
| `manifest/writes-module-entrypoint` | A versioned module reference: kind `module`, source, name, replacement of an earlier entrypoint, and no legacy output. | 120 s | None |
| `manifest/writes-builtin-legacy-runtimes` | Each runtime method matches `withLegacyRuntime`. Explicit, default, and loaded values, and version normalization. | 120 s | None |
| `manifest/removes-legacy-dependencies` | Named removal, unnamed-source removal, list replacement, and clearing. Other dependencies and the original builder stay intact. | 120 s | None |
| `manifest/regenerates-manifests` | Stable `generate(ws, lock, legacyJson)`, repeated generation, and load then regenerate in a nested workspace. | 120 s | None |
| `manifest/rejects-invalid-manifests` | `validate`, loading, and source validation. Missing fields, bad entrypoints, and target-version incompatibility. | 120 s | None |
| `manifest/rejects-invalid-manifest-files` | Malformed input, wrong field types, and format-specific fields. Errors start with the manifest filename. | 120 s | None |
| `omits-default-module-source` | A module source of `.` is not written to TOML. | 120 s | None |
| `keeps-loaded-module-source-unset` | Regeneration adds no module source. | 120 s | None |
| `keeps-workspace-manifest-fields` | Loaded fields that the caller does not set remain. | 120 s | None |
| `controls-loaded-jsonpins` | Loaded JSON pins are written only with `lock: true`. | 120 s | None |
| `controls-loaded-tomlpins` | Loaded TOML pins are written only with `lock: true`. | 120 s | None |
| `skips-pin-lookup` | Generation does not look up missing pins. | 120 s | None |
| `places-local-dependencies` | Local paths are relative to `ws.cwd` and have no pins. | 120 s | None |
| `replaces-dependency` | A dependency with the same name is replaced without a duplicate. | 120 s | None |
| `reuses-dependency-with-same-source` | A loaded dependency with the same source keeps its alias and JSON settings. | 120 s | None |
| `writes-custom-legacy-runtime` | A custom runtime source is written as given. | 120 s | None |
| `keeps-loaded-custom-legacy-runtime` | A loaded custom runtime and its pin remain until replaced. | 120 s | None |
| `rejects-bare-custom-legacy-runtime` | An unknown single-word runtime is rejected with the local path to use. | 120 s | None |
| `rejects-absolute-dependency-paths` | Absolute dependency sources are rejected when loaded. | 120 s | None |
| `controls-legacy-jsonoutput` | `legacyJson: false` removes JSON and retains TOML. | 120 s | None |
| `composes-custom-legacy-runtime` | A custom runtime composes with the other setters. `withLegacyInclude` adds and `withoutLegacyFields` resets. | 300 s | `64645f1967d3dba6fce951dd61ae4acd8d9b0861` |
| `honors-supplied-git-module` | A supplied commit is written only with `lock: true`. | 300 s | `64645f1967d3dba6fce951dd61ae4acd8d9b0861` |
| `places-same-repository-git-modules` | A module from the workspace repository is a relative path without a pin. Another commit stays a URL. | 300 s | `64645f1967d3dba6fce951dd61ae4acd8d9b0861`, `41fd4d3062d75a836429c6db5c42b4c6c9178ca4` |
| `e-2-e/legacy-caller-check` | A module that declares `v0.21.9` calls this checkout. | 180 s | None |

All Git downloads come from `github.com/dagger/sdk-helpers`. No check downloads
a CLI or builds Go code.

### Telemetry

A named check runs in the trace of its own `dagger check` command. Each helper
call in that check stays in the same trace. No check starts a nested CLI or
session.

These statements are expectations. No exported trace has confirmed these
expectations.

Read a result with these commands:

```sh
dagger trace <trace-id> --check <check-name>
dagger trace <trace-id> --check <check-name> --log -o check.log
```

A failed check must identify the failing call in its trace. A missing check span
is a test infrastructure bug.

### Go SDK coverage

This repository no longer compiles a generated Go module against the candidate
helper on each pull request. The nested check that did so was removed because
trace `b8b7c2110de2ddf450e8f6c2d76c51bb` was cancelled at 30 minutes with exit
137. That trace has inner CLI, connection, and subscription spans, but no SDK
check or `generateScope` spans. The trace does not establish why the inner
command stalled.

`dagger/go-sdk` commit `7eecc79efbbb8caf638aa28fcc5c34a7698ed5aa` has
`ManifestChecks.localClientCheck` and `gitClientCheck`, which cover generation
and compilation. Those checks use the helper dependency that go-sdk configures.
They do not test a pull request in this repository automatically.
