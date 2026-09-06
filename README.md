# SDK helpers

A pure Dang module with shared helpers for Dagger SDKs.

Add `github.com/dagger/sdk-helpers@main` as a dependency named `sdk-helpers`.
For a fixed version, use a full commit hash instead of `main`.

```dang
sdkHelpers.moduleManifest
  .withName("payments")
  .withDangEntrypoint("./main.dang")
  .withLegacyGoRuntime(moduleSource: "./src", engineVersion: "v0.21.9")
  .directory
```

`moduleManifest()` returns a `ModuleManifest` builder. Builder operations return
a new value. The example produces `dagger-module.toml` and `dagger.json`.
Use `.tomlFile` or `.legacyJSONFile` to return one file.

Load files with `moduleManifest(loadToml: file)` or
`moduleManifest(loadJson: file)`. If both are supplied, JSON loads first.
TOML sets the shared fields. Each format keeps its supported extra fields.
The loaders use Dang's TOML and JSON functions and Dagger's JSON inspection API.
They do not use a helper container or custom native code.

Manifest field names are case-sensitive. Unknown fields are ignored.
For example, use `include`; `Include` is an unknown field.
Load errors identify the file. Field type errors also identify the field and
expected type.

Adding a dependency sorts dependencies by name, or by source when the name is
empty. Equal sort keys keep their input order. Replacing a dependency keeps its
position.

## Checks

Run from this repository with the Dagger CLI and a working engine:

```sh
dagger check
```

The workspace installs `.dagger/modules/sdk-helpers-dev`. Each test is a separate
Dang function with a `Workspace` argument and an `@check` annotation.
The dev module calls `sdk-helpers` as a dependency.

List checks or run one check:

```sh
dagger check -l
dagger check sdk-helpers-dev:test-load-both
```

The suite covers all 93 saved comparison cases. It also checks the public API,
independent builder branches, field case, clear type errors, stable dependency
order, and sorting 100 loaded dependencies.

`.dagger/modules/sdk-helpers-dev/expected.json` contains the expected file bytes.
These bytes came from the former Go builder at Dagger commit `c1438d7fb`, in
`core/module_manifest.go`. `${ENGINE_VERSION}` stands for the running engine
version without build metadata. The checks compare file contents, names, and
mode `0644`. They cover both formats, all runtimes, version handling,
dependencies, escaping, and validation.

Review changes to expected results as API changes. The checks do not update them.
