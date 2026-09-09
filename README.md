# SDK helpers

Use `ModuleManifest` to load, change, and generate Dagger module manifests.
The manifest selects engine API version `v1.0.0`. Checks pass on engine
`v1.0.0-beta.11`. Earlier beta releases have not been tested.
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

## Add a dependency

Call `manifest.withLegacyRuntimeDependency(module)` with a `ModuleSource`.
The helper uses `module.moduleOriginalName` as the dependency name.

For Git dependencies, it writes `module.asString` as the source without changes.
This includes any `@version` or `@commit` present in that value. The engine can
add a version when it loads a source that had no version.

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

## Update existing SDK callers

- Replace `withLegacyRuntimeDependency(source, name, pin)` with
  `withLegacyRuntimeDependency(module)`.
- Replace `tomlFile`, `legacyJSONFile`, and `directory` with
  `generate(ws, ...)`. It returns a `Workspace`.
- Use `lock: true` when the SDK must write pins.

## Run checks

```sh
dagger check
```

The workspace runs the manifest checks and the checks in
`.dagger/modules/e2e`. These include a caller that selects the `v0.21.9` API
and Go SDK generation against this checkout.

The Git checks fetch fixed commits. The Go SDK check applies
`.dagger/modules/e2e/go-sdk.patch` to its fixed SDK commit in an isolated
container. The patch updates the SDK to the new helper API and adds its lock
option. It does not change the upstream SDK repository.
