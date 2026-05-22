# QmlSharp Documentation

QmlSharp is a managed framework and toolchain for building Qt/QML applications in C#. Views, ViewModels, and application logic are written through a fluent, QML-style C# DSL. The toolchain compiles those declarations into QML files, schema contracts, native QObject glue code, source maps, and build outputs. A thin Qt host loads and runs the generated application.

## Why QmlSharp?

Qt/QML is a mature, GPU-accelerated UI framework, but its development model requires hand-writing QML, managing JavaScript bindings, and bridging to native C++ for business logic. QmlSharp replaces that workflow:

- **C# is the single source of truth.** Views, ViewModels, and application logic live in one language with full IDE support, type checking, and refactoring.
- **The compiler generates everything else.** QML files, `.schema.json` runtime contracts, native QObject bindings, module metadata, and source maps are derived artifacts.
- **Interop is flat C ABI + P/Invoke.** No COM, no C++/CLI, no Node.js runtime. The native side is a thin Qt shell.
- **Qt 6.11 is the runtime baseline.** The toolchain discovers the Qt SDK through the `QT_DIR` environment variable.

## Documentation Overview

### Concepts

- [Architecture Overview](concepts/overview.md) -- High-level architecture and module map
- [Fluent DSL](concepts/fluent-dsl.md) -- The C# DSL for declaring QML views
- [ViewModels](concepts/viewmodels.md) -- State, commands, and effects
- [QML Generation](concepts/qml-generation.md) -- How C# becomes QML
- [Schema Contracts](concepts/schema-contracts.md) -- The `.schema.json` runtime contract model
- [Native Host](concepts/native-host.md) -- The Qt/C++ shell and managed/native boundary
- [Build Pipeline](concepts/build-pipeline.md) -- The 8-stage build system
- [Dev Tools](concepts/dev-tools.md) -- File watching, hot reload, REPL, profiling

### Reference

- [Managed API](reference/managed-api.md) -- Public C# APIs by module
- [Native ABI](reference/native-abi.md) -- The flat C ABI surface
- [CLI](reference/cli.md) -- `dotnet qmlsharp` command reference
- [Generated Artifacts](reference/generated-artifacts.md) -- Output files and their formats
- [Diagnostics](reference/diagnostics.md) -- Compiler and build diagnostic codes
- [Configuration](reference/configuration.md) -- `qmlsharp.json` reference

### Contributing

- [Development](contributing/development.md) -- Building and running from source
- [Testing](contributing/testing.md) -- Test organization and conventions
- [CI](contributing/ci.md) -- Continuous integration setup
- [Qt SDK](contributing/qt-sdk.md) -- Qt installation and `QT_DIR`

## Project Status

QmlSharp is in early development. The compiler, runtime contracts, and public APIs are stabilizing. The current implementation covers:

- Qt type registry with full qmltypes/qmldir/metatypes parsing
- QML AST with builders, validation, traversal, and transforms
- QML emitter with source maps and configurable formatting
- Qt toolchain wrappers (qmllint, qmlformat, qmltc, rcc, and more)
- Fluent C# DSL with DispatchProxy-based builder generation
- DSL code generator for type-safe Qt type wrappers
- Compiler pipeline (analysis, extraction, transformation, post-processing, emission)
- Runtime host with instance registry, state sync, command routing, and effect dispatch
- 8-stage build pipeline with C++ code generation, native builds, and product layout
- Dev tools (file watcher, dev console, hot reload, REPL, performance profiler, error overlay)
- CLI (`dotnet qmlsharp build`, `dev`, `doctor`, `init`, `clean`)
- CI on Windows, Linux, and macOS

## License

[MIT](https://github.com/nicksoftware/QmlSharp/blob/main/LICENSE)

---

*This documentation site is built from plain Markdown under `docs/`. GitHub Pages publishing can be configured to serve from this directory or from a generated static site.*
