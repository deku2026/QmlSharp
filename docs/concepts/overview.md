# Architecture Overview

QmlSharp is organized into a layered set of .NET libraries, a native C/C++ host, and a CLI. Each layer has a clear responsibility and communicates through well-defined contracts.

## Module Map

```
CLI layer
  QmlSharp.Cli ........... dotnet qmlsharp command-line tool

Orchestration layer
  QmlSharp.Build ......... 8-stage build pipeline, C++ codegen, product layout
  QmlSharp.DevTools ...... file watcher, hot reload, REPL, profiler, dev server

Runtime layer
  QmlSharp.Host .......... managed engine facade, instance registry, state sync,
                           command routing, effect dispatch, hot reload coordinator
  QmlSharp.Compiler ...... C# analysis, ViewModel extraction, DSL transformation,
                           post-processing, QML emission, source maps

Generation layer
  QmlSharp.Dsl ........... fluent QML-style C# DSL (IObjectBuilder, ObjectFactory)
  QmlSharp.Dsl.Generator . type-safe DSL wrapper generation from Qt type registry

Infrastructure layer
  QmlSharp.Qml.Emitter .. QML AST to text, source map tracking, emit options
  QmlSharp.Qml.Ast ....... immutable QML AST nodes, builders, validation, traversal
  QmlSharp.Qt.Tools ...... wrappers for qmllint, qmlformat, qmltc, rcc, and others
  QmlSharp.Registry ...... unified Qt type registry (qmltypes, qmldir, metatypes)
  QmlSharp.Core .......... shared value types (QmlColor, QmlFont, Vector2/3/4, etc.)

Native layer
  native/ ................ C/C++ Qt shell: QQmlEngine lifecycle, QObject type
                           registration, flat C ABI for P/Invoke interop
```

## Data Flow

A typical compile-and-run cycle:

```
C# source (.cs)
  |
  v
[CSharpAnalyzer] ---- Roslyn analysis, View/ViewModel discovery
  |
  v
[ViewModelExtractor] - @State, @Command, @Effect extraction, schema generation
  |
  v
[DslTransformer] ----- fluent DSL calls -> QML AST nodes
  |
  v
[PostProcessor] ------ import injection, lifecycle hooks, runtime glue
  |
  v
[QmlEmitter] --------- QML AST -> .qml text + source maps
  |
  v
[CompilerOutputWriter] .qml files, .schema.json, event-bindings.json, source maps
  |
  v
[BuildPipeline] ------ C++ codegen, CMake build, qmldir/qmltypes, product layout
  |
  v
[QmlSharpEngine] ----- native init, type registration, QML load, event loop
```

## Key Design Decisions

**Immutable data throughout.** AST nodes, schemas, compilation results, and registry entries are all immutable records. This makes the compiler pipeline deterministic and thread-safe.

**Schema-driven code generation.** The `ViewModelSchema` is the single contract between the compiler and all downstream consumers: the QML emitter, the C++ code generator, the native type registrar, and the runtime host.

**Flat C ABI.** The managed/native boundary uses a small set of `extern "C"` functions with `__cdecl` calling convention. String ownership follows a simple rule: the caller passes `const char*`, and any string returned by the native side must be freed with `qmlsharp_free_string`.

**Registry-based type safety.** The Qt type registry (`QmlRegistry`) is built from three Qt metadata formats (qmltypes, qmldir, metatypes) and provides O(1) lookups by qualified name, module URI, inheritance chain, property name, signal name, and method name. The DSL generator uses this registry to produce type-safe C# builder interfaces for every Qt type.
