# Schema Contracts

The `.schema.json` file is QmlSharp's central contract between the compiler, the runtime host, the C++ code generator, and the native type registrar. Every ViewModel produces one schema that describes its full runtime interface.

## ViewModelSchema

The compiler produces a `ViewModelSchema` for each ViewModel class. This record carries all the information needed to:

- Generate QML property declarations and signal handlers
- Generate C++ QObject subclasses with Q_PROPERTY/Q_INVOKABLE
- Register types with the QML engine at runtime
- Route commands and dispatch effects in the managed host

### Schema Fields

| Field | Description |
|-------|-------------|
| `schemaVersion` | Contract version string (currently `"1.0"`) |
| `className` | C# ViewModel class name |
| `moduleName` | QML module name |
| `moduleUri` | QML module URI (e.g., `"QmlSharp.MyApp"`) |
| `moduleVersion` | Major.minor version |
| `version` | Schema revision counter |
| `compilerSlotKey` | Deterministic hash for native type registration |
| `properties` | Array of state property entries |
| `commands` | Array of command entries |
| `effects` | Array of effect entries |
| `lifecycle` | Lifecycle capability flags |

### State Entry

```json
{
  "name": "count",
  "type": "int",
  "defaultValue": "0",
  "readOnly": false,
  "memberId": 12345
}
```

### Command Entry

```json
{
  "name": "increment",
  "parameters": [
    { "name": "amount", "type": "int" }
  ],
  "commandId": 67890
}
```

### Effect Entry

```json
{
  "name": "onCountChanged",
  "payloadType": "int",
  "effectId": 11111,
  "parameters": [
    { "name": "next", "type": "int" }
  ]
}
```

## Event Bindings

The `event-bindings.json` file is a project-level manifest that aggregates all command and effect bindings across ViewModels. It is used by the runtime host for fast command/effect routing without re-parsing individual schemas.

```json
{
  "schemaVersion": "1.0",
  "commands": [
    {
      "viewModelClass": "CounterViewModel",
      "commandName": "increment",
      "commandId": 67890,
      "parameterTypes": ["int"]
    }
  ],
  "effects": [
    {
      "viewModelClass": "CounterViewModel",
      "effectName": "onCountChanged",
      "effectId": 11111,
      "payloadType": "int"
    }
  ]
}
```

## Schema Serialization

Schemas are serialized to JSON by `ViewModelSchemaSerializer` with deterministic formatting (sorted keys, consistent indentation, LF line endings). This ensures that unchanged ViewModels produce byte-identical schema files across builds.

## Schema Diffing

The `SchemaDiffer` in the dev-tools module compares schema versions to detect changes that require a full rebuild versus changes that can be hot-reloaded. Property additions, type changes, and command signature changes trigger a full rebuild; value-only changes can be applied incrementally.

## How Schemas Drive Code Generation

```
ViewModelSchema
  |
  +---> QML emitter: property declarations, signal handlers
  |
  +---> C++ code generator: QObject subclass (.h/.cpp)
  |
  +---> qmldir generator: module type registration
  |
  +---> qmltypes generator: QML tooling metadata
  |
  +---> Runtime host: type registration, state sync metadata
```

## Compiler Slot Key

The `compilerSlotKey` is a deterministic hash derived from the ViewModel's module URI, version, and class name. It uniquely identifies a generated native type across the managed/native boundary and survives incremental rebuilds as long as the ViewModel's identity does not change.
