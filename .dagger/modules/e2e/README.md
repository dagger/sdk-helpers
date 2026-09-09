# Integration checks

The root `dagger.toml` registers this module. Run all checks with `dagger check`.

`legacyCallerCheck` calls the public helper from a module that declares
`v0.21.9`. The helper keeps its own `v1.0.0` API selection.

`goSdkCheck` runs the Go SDK in an isolated container. It uses SDK commit
`fef4ddfc4d6ec0ffa2c0aa3361f5466d800d4069` and the helper source from this
workspace. `go-sdk.patch` updates the SDK caller to `withLegacyRuntimeDependency(module)`
and `generate(ws, lock)`. The patch also adds an SDK lock option for this test.

The inner checks cover:

- A Go module with a local client in a nested directory.
- Both manifest formats, with relative local paths and no default pins.
- Generated module and standalone client files.
- Repeated generation without manifest changes.
- Compilation of a Go function that calls the local dependency.
- A Git client with default output and with locking enabled.

The container uses the `v1.0.0-beta.11` Linux CLI with access to the running
test engine. The first run downloads the CLI, SDK source, and build images.

When the SDK adopts this API, update the fixed SDK commit and remove the patch.
