# Development

This page covers how to build and run QmlSharp from source.

## Prerequisites

- **.NET SDK** -- version specified in `global.json` (currently .NET 10)
- **Qt 6.11.0** -- `qtbase` and `qtdeclarative` modules
- **CMake** -- for building the native library
- **C++ compiler** -- MSVC (Windows), GCC (Linux), or Clang (macOS)

Set the `QT_DIR` environment variable to your Qt platform directory:

```bash
# Windows (PowerShell)
$env:QT_DIR = "<path-to-qt-6.11.0-kit>"

# Linux / macOS
export QT_DIR="/opt/Qt/6.11.0/gcc_64"
```

## Repository Structure

```
QmlSharp/
  src/
    QmlSharp.Core/          Shared value types
    QmlSharp.Registry/      Qt type registry
    QmlSharp.Qml.Ast/       QML abstract syntax tree
    QmlSharp.Qml.Emitter/   QML code emitter
    QmlSharp.Qt.Tools/      Qt toolchain wrappers
    QmlSharp.Dsl/            Fluent C# DSL
    QmlSharp.Dsl.Generator/  DSL code generator
    QmlSharp.Compiler/       C# to QML compiler
    QmlSharp.Host/           Runtime host
    QmlSharp.Build/          Build pipeline
    QmlSharp.DevTools/       Development tools
    QmlSharp.Cli/            CLI entry point
  native/
    include/qmlsharp/       Public C ABI headers
    src/                    Native implementation
    tests/                  Native tests
    windows/                Windows MSBuild projects
    CMakeLists.txt          CMake build
  tests/
    QmlSharp.*.Tests/       Per-module test projects
    QmlSharp.Integration.Tests/  Cross-module integration tests
    fixtures/               Test fixture projects
  tools/
    GenerateRegistrySnapshot/   Registry snapshot generator
  templates/
    qmlsharp-app/           Application template
    qmlsharp-library/       Library template
  docs/                     Documentation (this site)
```

## Solution Files

| File | Description |
|------|-------------|
| `QmlSharp.slnx` | .NET-only solution for all managed projects |
| `QmlSharp.Win.slnx` | Windows solution including native MSBuild projects |

## Building

### .NET Projects

```bash
dotnet build QmlSharp.slnx
```

### Windows (Full Solution)

```powershell
msbuild QmlSharp.Win.slnx /m /p:Configuration=Debug /p:Platform=x64
```

This builds both the .NET projects and the native library (via MSBuild/vcxproj).

### Native Library (CMake)

```bash
cd native
cmake -B build -DCMAKE_PREFIX_PATH="$QT_DIR"
cmake --build build
```

## Running Tests

### .NET Tests

```bash
dotnet test QmlSharp.slnx --configuration Debug --verbosity normal
```

### Native Tests

```bash
./build/debug/bin/qmlsharp_native_tests
./build/debug/bin/qmlsharp_native_performance_tests
```

### Formatting Check

```bash
dotnet format QmlSharp.slnx --verify-no-changes --no-restore
```

## Code Style

- C# follows the project's `.editorconfig` and analyzer rules (`Meziantou.Analyzer`)
- All warnings are treated as errors in CI
- Formatting is checked with `dotnet format`
- C++ follows Qt conventions for the native shell
