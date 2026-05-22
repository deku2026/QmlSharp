# Generated Artifacts

QmlSharp produces several categories of generated files during compilation and build. This page documents each artifact type, its format, and its purpose.

## Compiler Artifacts

These are produced by the QmlSharp compiler (`QmlSharp.Compiler`).

### QML Files (`.qml`)

Generated from C# View declarations. Each View class produces one QML file containing the translated UI tree.

```
dist/qml/<ModuleUri>/<ViewName>.qml
```

QML files include:
- Import statements (auto-injected by the compiler)
- Root object with property bindings, signal handlers, and children
- Lifecycle hooks (`Component.onCompleted`, `Component.onDestruction`)
- Connection objects for effect listeners

### Schema Files (`.schema.json`)

One per ViewModel. Contains the complete runtime contract:

```
dist/schemas/<ViewModelClassName>.schema.json
```

Format:

```json
{
  "schemaVersion": "1.0",
  "className": "CounterViewModel",
  "moduleName": "Counter",
  "moduleUri": "QmlSharp.MyApp",
  "moduleVersion": { "major": 1, "minor": 0 },
  "compilerSlotKey": "a1b2c3d4",
  "properties": [ ... ],
  "commands": [ ... ],
  "effects": [ ... ],
  "lifecycle": { "onMounted": true, "onUnmounting": false, "hotReload": true }
}
```

### Event Bindings (`event-bindings.json`)

Project-level manifest aggregating all command and effect bindings:

```
dist/event-bindings.json
```

Used by the runtime host for fast command/effect routing. See [Schema Contracts](../concepts/schema-contracts.md) for the format.

### Source Maps (`.qml.map`)

One per QML file. Maps generated QML positions back to C# source positions:

```
dist/source-maps/<ViewName>.qml.map
```

Format:

```json
{
  "schemaVersion": "1.0",
  "sourceFilePath": "src/CounterView.cs",
  "outputFilePath": "dist/qml/QmlSharp/MyApp/CounterView.qml",
  "mappings": [
    {
      "outputLine": 5,
      "outputColumn": 1,
      "sourceFilePath": "src/CounterView.cs",
      "sourceLine": 12,
      "sourceColumn": 9,
      "symbol": "count",
      "nodeKind": "Binding"
    }
  ]
}
```

All positions are 1-based.

## Build Artifacts

These are produced by the build pipeline (`QmlSharp.Build`).

### Module Metadata

#### `qmldir`

QML module directory file listing registered types:

```
dist/qml/<ModuleUri>/qmldir
```

#### `.qmltypes`

Qt tooling metadata for IDE support and validation:

```
dist/qml/<ModuleUri>/plugins.qmltypes
```

### C++ Generated Code

#### QObject Headers and Implementations

One `.h`/`.cpp` pair per ViewModel, containing:
- `Q_OBJECT` macro
- `Q_PROPERTY` declarations for each `[State]` property
- `Q_INVOKABLE` declarations for each `[Command]` method
- Signal declarations used by `[Effect]` routing
- Property getters/setters plus typed `setPropertyFromManaged*` entry points for managed state sync

#### `type_registration.cpp`

Auto-generated code that calls `qmlsharp_register_module` for all ViewModel types.

#### `CMakeLists.txt`

Generated CMake project for building the native library. Links against Qt6::Quick, Qt6::Qml, and the handwritten ABI sources.

### Native Library

Compiled from generated C++ and the handwritten ABI:

| Platform | Path |
|----------|------|
| Windows | `dist/native/qmlsharp_native.dll` |
| Linux | `dist/native/libqmlsharp_native.so` |
| macOS | `dist/native/libqmlsharp_native.dylib` |

### Qt Resource Collection (`.qrc`)

Generated when assets are bundled:

```
dist/resources.qrc
```

### Product Manifest (`manifest.json`)

Build metadata and file integrity hashes:

```
dist/manifest.json
```

Format:

```json
{
  "projectName": "MyApp",
  "version": "0.1.0",
  "buildMode": "development",
  "buildTimestamp": "2026-05-07T12:00:00Z",
  "qtVersion": "6.11.0",
  "dotNetVersion": "10.0",
  "qmlModules": ["QmlSharp.MyApp"],
  "viewModels": ["CounterViewModel"],
  "fileHashes": { ... },
  "nativeLib": "native/qmlsharp_native.dll",
  "managedAssembly": "MyApp.dll"
}
```

## Output Directory Layout

```
dist/
  qml/
    <ModuleUri>/
      *.qml
      qmldir
      plugins.qmltypes
  schemas/
    *.schema.json
  event-bindings.json
  source-maps/
    *.qml.map
  native/
    qmlsharp_native.dll (or .so / .dylib)
    generated/
      *.h, *.cpp
      type_registration.cpp
      CMakeLists.txt
  managed/
    <ProjectName>.dll
  assets/
  manifest.json
```

## Planned Artifacts

The following artifacts are planned but may not be fully implemented yet:

- **AOT-compiled QML** (`qmlcachegen` output) -- generated when `build.aot` is enabled
- **Platform distribution packages** -- platform-specific packaging for deployment
