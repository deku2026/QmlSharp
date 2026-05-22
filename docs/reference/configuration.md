# Configuration Reference

QmlSharp projects are configured through a `qmlsharp.json` file in the project root. This file controls the compiler, build pipeline, native build, and development tools.

## Example

```json
{
  "entry": "./src/Program.cs",
  "outDir": "./dist",
  "qt": {
    "dir": null,
    "modules": ["QtQuick", "QtQuick.Controls"]
  },
  "build": {
    "aot": false,
    "lint": true,
    "format": true,
    "sourceMaps": true,
    "incremental": true,
    "mode": "development"
  },
  "native": {
    "prebuilt": false,
    "cmakePreset": "default"
  },
  "dev": {
    "hotReload": true,
    "watchPaths": ["./src"],
    "debounceMs": 200
  },
  "module": {
    "prefix": "QmlSharp.MyApp",
    "version": { "major": 1, "minor": 0 }
  },
  "name": "MyApp",
  "version": "0.1.0"
}
```

## Top-Level Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `entry` | `string?` | `null` | Path to the C# entry point file |
| `outDir` | `string` | `"./dist"` | Output directory for build products |
| `name` | `string?` | `null` | Project name (used in manifest and diagnostics) |
| `version` | `string` | `"0.0.0"` | Project version string |
| `qt` | object | -- | Qt SDK configuration (required) |
| `build` | object | -- | Build behavior options |
| `native` | object | -- | Native C++ build configuration |
| `dev` | object | -- | Development mode settings |
| `module` | object | -- | QML module identity (required) |

## `qt` -- Qt SDK Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `dir` | `string?` | `null` | Path to the Qt installation. When `null`, the build system reads from the `QT_DIR` environment variable |
| `modules` | `string[]` | `["QtQuick", "QtQuick.Controls"]` | Qt modules required by this project |

### Qt SDK Discovery

The Qt SDK is resolved in this order:

1. `qt.dir` in `qmlsharp.json`
2. `QT_DIR` environment variable

The path should point to the Qt platform directory for Qt 6.11.0, such as a Windows `msvc2022_64` kit directory or a Linux `gcc_64` kit directory.

## `build` -- Build Behavior

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `aot` | `bool` | `false` | Enable AOT QML compilation via `qmlcachegen` |
| `lint` | `bool` | `true` | Run `qmllint` on generated QML files |
| `format` | `bool` | `true` | Run `qmlformat` on generated QML files |
| `sourceMaps` | `bool` | `true` | Generate C#-to-QML source maps |
| `incremental` | `bool` | `true` | Enable file-level incremental compilation |
| `mode` | `string` | `"development"` | Build mode: `"development"` or `"production"` |

## `native` -- Native Build

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `prebuilt` | `bool` | `false` | Skip native build and use an existing native library |
| `cmakePreset` | `string` | `"default"` | CMake preset name for the native build |

When `prebuilt` is `true`, the build pipeline expects a pre-compiled native library at the standard output location and skips Build Stage 7.

## `dev` -- Development Mode

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `hotReload` | `bool` | `true` | Enable hot reload during development |
| `watchPaths` | `string[]` | `["./src"]` | Directories to watch for file changes |
| `debounceMs` | `int` | `200` | Debounce interval for file change events in milliseconds |

## `module` -- QML Module Identity

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `prefix` | `string` | -- | QML module URI prefix (required) |
| `version` | object | `{ "major": 1, "minor": 0 }` | Module version |

The `prefix` determines the QML import URI. For example, with `"prefix": "QmlSharp.MyApp"`, generated types are registered under:

```qml
import QmlSharp.MyApp 1.0
```

## Validation

The `IConfigLoader` validates configuration on load. Invalid configuration produces a `ConfigParseException` with diagnostic details including the field path and reason.

`dotnet qmlsharp doctor` includes configuration validation as one of its diagnostic checks.
