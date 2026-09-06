# SDK helpers

A Dang module with shared helpers for Dagger SDKs.

## Manifest builder

The API provides one additive builder. It can create a manifest or load existing TOML and JSON manifests.

```graphql
extend type Query {
  moduleManifest(
    loadTOML: FileID
    loadJSON: FileID
  ): ModuleManifest!
}

type ModuleManifest implements Node {
  id: ID!

  withName(name: String!): ModuleManifest!
  withLegacyRuntimeDependency(
    source: String!
    name: String
    pin: String
  ): ModuleManifest!
  withoutLegacyRuntimeDependency(name: String!): ModuleManifest!
  withoutLegacyRuntimeDependencies: ModuleManifest!

  withDangEntrypoint(source: String!): ModuleManifest!
  withModuleEntrypoint(source: String!): ModuleManifest!

  withLegacyGoRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyDangRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyPythonRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyTypescriptRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyPHPRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyElixirRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyJavaRuntime(moduleSource: String, engineVersion: String): ModuleManifest!
  withLegacyInclude(path: String!): ModuleManifest!
  withoutLegacyFields: ModuleManifest!

  validate(targetEngineVersion: String): Void!
  tomlFile: File!
  legacyJSONFile: File!
  directory: Directory!
}
```

A fat manifest can contain an entrypoint and legacy runtime fields. New engines use the entrypoint. Older engines use the legacy runtime. The manifest has no explicit version field.
