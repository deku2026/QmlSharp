# Qt SDK Setup

QmlSharp requires Qt 6.11.0 with the `qtbase` and `qtdeclarative` modules. This page covers installation and configuration.

## Required Modules

| Module | Provides |
|--------|----------|
| `qtbase` | `QGuiApplication`, `QObject`, core Qt types |
| `qtdeclarative` | `QQmlEngine`, QML runtime, Qt Quick |

## Installation

### Qt Online Installer

Download from [qt.io](https://www.qt.io/download-qt-installer-oss) and install Qt 6.11.0 with the appropriate compiler kit:

| Platform | Kit |
|----------|-----|
| Windows | `msvc2022_64` |
| Linux | `gcc_64` |
| macOS | `macos` |

### Manual Archive Download

For CI or headless setups, download archives directly from `download.qt.io`:

```
https://download.qt.io/online/qtsdkrepository/windows_x86/desktop/qt6_6110/
```

Extract `qtbase` and `qtdeclarative` archives to a directory and point `QT_DIR` there.

## Configuration

Set the `QT_DIR` environment variable to the Qt platform root. This is the directory that contains `bin/`, `lib/`, `include/`, and `lib/cmake/Qt6/`.

### Windows (PowerShell)

```powershell
$env:QT_DIR = "<path-to-qt-6.11.0-kit>"
```

### Linux

```bash
export QT_DIR="/opt/Qt/6.11.0/gcc_64"
```

### macOS

```bash
export QT_DIR="$HOME/Qt/6.11.0/macos"
```

### In qmlsharp.json

Alternatively, set the Qt path in the project configuration:

```json
{
  "qt": {
    "dir": "/path/to/Qt/6.11.0/gcc_64"
  }
}
```

The `qt.dir` field takes precedence over `QT_DIR` when both are set.

## Verification

Run `dotnet qmlsharp doctor` to verify the Qt installation:

```bash
dotnet qmlsharp doctor
```

Or check manually:

```bash
$QT_DIR/bin/qmllint --version
$QT_DIR/bin/qmlformat --version
```

## CMake Integration

The native build uses CMake to find Qt. The build pipeline sets `CMAKE_PREFIX_PATH` to `QT_DIR` automatically. For manual native builds:

```bash
cmake -B build -DCMAKE_PREFIX_PATH="$QT_DIR"
cmake --build build
```

## Qt Modules Used by QmlSharp

The default project template links against:

- `Qt6::Quick` -- Qt Quick rendering and scene graph
- `Qt6::Qml` -- QML engine and type system

Additional modules can be specified in `qmlsharp.json` under `qt.modules`.

## Platform Notes

### Windows

The Windows CI uses MSVC 2022 (x64). The Windows SDK `rc.exe` must be on PATH for resource compilation. The CI workflow detects this automatically.

### Linux

Qt must be built with GCC. The offscreen QPA platform (`QT_QPA_PLATFORM=offscreen`) is used in CI for headless testing.

### macOS

Both arm64 and x64 are supported through the platform distributor. Use the appropriate Qt kit for your architecture.
