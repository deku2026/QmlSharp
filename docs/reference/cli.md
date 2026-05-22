# CLI Reference

QmlSharp provides a command-line tool invoked as `dotnet qmlsharp`. It supports five commands for building, developing, diagnosing, scaffolding, and cleaning projects.

## Common Project Option

The project-scoped commands accept this option:

| Option | Description |
|--------|-------------|
| `--project-dir <path>` | Path to the QmlSharp project directory (default: current directory) |

## `dotnet qmlsharp build`

Runs the full 8-stage build pipeline.

```
dotnet qmlsharp build [options]
```

| Option | Description |
|--------|-------------|
| `--force` | Force full rebuild, ignoring incremental cache |
| `--files <glob>` | Only compile files matching this glob pattern |
| `--dry-run` | Validate config and report what would change without building |
| `--json` | Output structured JSON progress and result |
| `--library` | Build as a reusable QmlSharp library |
| `--project-dir <path>` | Project directory |

**Exit codes:**

| Code | Meaning |
|------|---------|
| `0` | Build succeeded |
| `1` | Build failed |
| `2` | Configuration or command error |
| `130` | Cancelled |

## `dotnet qmlsharp dev`

Starts a development session with file watching, hot reload, and interactive REPL.

```
dotnet qmlsharp dev [options]
```

| Option | Description |
|--------|-------------|
| `--headless` | Run without launching the Qt window |
| `--entry <path>` | Override the entry point file |
| `--project-dir <path>` | Project directory |

The dev server:
1. Loads configuration from `qmlsharp.json`
2. Runs an initial build
3. Launches the Qt application (unless `--headless`)
4. Watches source files for changes
5. Triggers incremental compilation and hot reload on change
6. Provides an interactive REPL (C# and QML modes)

## `dotnet qmlsharp doctor`

Runs environment diagnostics to check for common configuration issues.

```
dotnet qmlsharp doctor [options]
```

| Option | Description |
|--------|-------------|
| `--fix` | Attempt to fix issues automatically |
| `--check <id>` | Run only a specific diagnostic check |
| `--project-dir <path>` | Project directory |

Checks include:
- Qt SDK availability and version
- .NET SDK version compatibility
- `qmlsharp.json` configuration validity
- Native toolchain availability (CMake, MSVC/GCC/Clang)
- Required Qt modules

## `dotnet qmlsharp init`

Scaffolds a new QmlSharp project from a template.

```
dotnet qmlsharp init [options]
```

| Option | Description |
|--------|-------------|
| `--template <name>` | Template name (default: `default`) |
| `--target-dir <path>` | Target directory for the new project (default: current directory) |

Available templates:

| Template | Description |
|----------|-------------|
| `qmlsharp-app` | Standalone application with View, ViewModel, and Program.cs |
| `qmlsharp-library` | Reusable library with View and ViewModel |

## `dotnet qmlsharp clean`

Removes build artifacts from the output directory.

```
dotnet qmlsharp clean [options]
```

| Option | Description |
|--------|-------------|
| `--cache` | Also clear the incremental compilation cache |
| `--project-dir <path>` | Project directory |
