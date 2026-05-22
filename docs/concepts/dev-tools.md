# Dev Tools

QmlSharp includes a suite of development tools for live feedback during development. These tools are implemented in the `QmlSharp.DevTools` module and orchestrated by the `dotnet qmlsharp dev` command.

## Dev Server

The `DevServer` is the top-level facade that coordinates all dev tools. It manages a state machine with these states:

| State | Description |
|-------|-------------|
| `Idle` | Not running |
| `Starting` | Initializing watcher, console, and profiler |
| `Building` | Running the initial build |
| `Running` | Application is running, watching for changes |
| `Reloading` | Applying a hot reload cycle |
| `Error` | Recoverable error, waiting for fix |
| `Stopping` | Shutting down |

Start a dev session:

```
dotnet qmlsharp dev
```

Options:

| Flag | Purpose |
|------|---------|
| `--headless` | Run without launching the Qt window |
| `--entry <path>` | Override the entry point |
| `--project-dir <path>` | Specify the project directory |

## File Watcher

The `FileWatcher` monitors source files for changes and triggers incremental rebuilds. It supports:

- Configurable watch paths (default: `./src`)
- File extension filters
- Debounced change events (default: 200ms)
- Polling fallback for file systems without native change notification

## Hot Reload

The `HotReloadOrchestrator` coordinates live updates:

1. Detects changed files via the file watcher
2. Runs incremental compilation on changed sources
3. Compares schemas to detect breaking changes (`SchemaDiffer`)
4. If schemas are compatible: reloads QML and rehydrates state
5. If schemas changed: triggers a full rebuild and restart

The reload protocol uses the native ABI's snapshot/reload/restore cycle:
- `qmlsharp_capture_snapshot` -- saves current instance state
- `qmlsharp_reload_qml` -- loads updated QML
- `qmlsharp_restore_snapshot` -- rehydrates state into new instances

The `InstanceMatcher` matches pre-reload instances to post-reload instances for state transfer.

## Dev Console

The `DevConsole` provides source-map-aware output formatting. Diagnostic messages include both the generated QML location and the original C# source location, resolved through the compiler's source maps via `SourceMapLookup`.

## REPL

The interactive `Repl` supports two modes:

- **C# mode** -- evaluates C# expressions using Roslyn scripting
- **QML mode** -- evaluates QML snippets

Built-in commands:
- History navigation
- Built-in commands for inspecting state, instances, and metrics

## Performance Profiler

The `PerfProfiler` collects timing data during builds and hot reload cycles. It exports traces in Chrome Trace Event format (`.json`), which can be loaded in `chrome://tracing` or Perfetto for visual analysis.

## Error Overlay

The `ErrorOverlay` displays compiler and runtime diagnostics directly in the QML application window during development. Errors are mapped back to C# source locations using source maps. The overlay is controlled through the native ABI (`qmlsharp_show_error` / `qmlsharp_hide_error`).

## Schema Differ

The `SchemaDiffer` compares two `ViewModelSchema` versions to produce a `SchemaDiffResult`. This determines whether a change can be hot-reloaded or requires a full rebuild:

- **Hot-reloadable**: value changes, non-breaking additions
- **Requires rebuild**: type changes, removed members, command signature changes
