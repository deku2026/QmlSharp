# Build Pipeline

QmlSharp's build system is an 8-stage pipeline that transforms a C# project into a deployable Qt/QML application. Each stage has a clear input, output, and set of diagnostics.

## Pipeline Overview

```
Stage 1: Config Loading
Stage 2: C# Compilation
Stage 3: Module Metadata Generation
Stage 4: Dependency Resolution
Stage 5: Asset Bundling
Stage 6: QML Validation
Stage 7: C++ Code Generation and Native Build
Stage 8: Output Assembly
```

## Stages

### Stage 1: Config Loading

Loads and validates `qmlsharp.json` from the project directory. Resolves Qt SDK path from the `QT_DIR` environment variable or the `qt.dir` config field. Produces a `QmlSharpConfig` and a `BuildContext`.

### Stage 2: C# Compilation

Compiles the C# project using `Microsoft.CodeAnalysis.Workspaces.MSBuild`. Runs the QmlSharp compiler pipeline (analysis, extraction, transformation, post-processing, emission) to produce `CompilationUnit` outputs with QML text, schemas, source maps, and event bindings.

### Stage 3: Module Metadata Generation

Generates Qt module metadata from ViewModel schemas:
- **qmldir** -- QML module directory file listing registered types
- **qmltypes** -- Qt tooling metadata for IDE support and validation

### Stage 4: Dependency Resolution

Resolves QmlSharp package dependencies:
- Discovers third-party QmlSharp libraries
- Collects QML import paths
- Merges external schemas

### Stage 5: Asset Bundling

Collects and bundles application assets:
- Discovers assets from configured paths
- Generates Qt resource collection files (`.qrc`)
- Prepares assets for embedding via `rcc`

### Stage 6: QML Validation

Runs Qt toolchain validation on generated QML:
- **qmllint** -- static analysis for QML correctness
- **qmlformat** -- code formatting (when `build.format` is enabled)
- Reports diagnostics with source-map-aware locations

### Stage 7: C++ Code Generation and Native Build

Generates and compiles native code:
- **C++ QObject subclasses** -- one `.h`/`.cpp` pair per ViewModel, with Q_PROPERTY, Q_INVOKABLE, and signal declarations
- **type_registration.cpp** -- QML type registration boilerplate
- **CMakeLists.txt** -- links against Qt6::Quick, Qt6::Qml, and the handwritten ABI sources
- **CMake build** -- configure and build the native library

Can be skipped with `native.prebuilt: true` when using a pre-compiled native library.

### Stage 8: Output Assembly

Assembles all artifacts into the output directory (`dist/` by default):
- Copies QML files, schemas, source maps, event bindings
- Copies the native library and managed assembly
- Generates `manifest.json` with build metadata and file hashes
- Validates the output directory structure

## Build Artifacts

The `BuildArtifacts` record tracks everything produced by the pipeline:

| Artifact | Description |
|----------|-------------|
| `QmlFiles` | Generated `.qml` files |
| `SchemaFiles` | `.schema.json` per ViewModel |
| `EventBindingsFile` | `event-bindings.json` manifest |
| `SourceMapFiles` | C#-to-QML source maps |
| `ModuleMetadataFiles` | `qmldir` and `.qmltypes` |
| `AssetFiles` | Bundled application assets |
| `QrcFile` | Qt resource collection |
| `NativeLibraryPath` | Compiled native library |
| `AssemblyPath` | Compiled C# entry assembly |

## Build Result

Each stage produces a `PhaseResult` with success status, duration, and diagnostics. The aggregate `BuildResult` includes:

- Per-phase results
- Aggregate diagnostics
- Build statistics (files compiled, schemas generated, C++ files generated, assets collected)
- Product manifest
- Build artifacts

## Incremental Builds

When `build.incremental` is enabled (the default), the compiler caches compilation results per file and only recompiles changed files. The build pipeline skips unchanged phases when their inputs have not changed.

## Product Layout

The `ProductLayout` manages the output directory structure:

```
dist/
  qml/
    <ModuleUri>/
      *.qml
  schemas/
    *.schema.json
  event-bindings.json
  source-maps/
    *.qml.map
  native/
    qmlsharp_native.dll (or .so / .dylib)
  managed/
    <ProjectName>.dll
  assets/
  manifest.json
```

## Platform Distribution

The `PlatformDistributor` packages the build output for deployment. It handles platform-specific concerns:

| Platform | Native Library | Qt Dependencies |
|----------|---------------|----------------|
| Windows x64 | `qmlsharp_native.dll` | Qt6Quick.dll, Qt6Qml.dll, etc. |
| Linux x64 | `libqmlsharp_native.so` | libQt6Quick.so, etc. |
| macOS arm64/x64 | `libqmlsharp_native.dylib` | QtQuick.framework, etc. |

## Build Configuration

See [Configuration](../reference/configuration.md) for the full `qmlsharp.json` reference.
