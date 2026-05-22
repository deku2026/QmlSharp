# Managed API Reference

This page documents the primary public C# APIs across QmlSharp's modules. APIs are grouped by user-facing concept rather than exhaustive method listing.

## QmlSharp.Core

Shared value types used across modules.

| Type | Description |
|------|-------------|
| `QmlColor` | RGBA color with string conversion |
| `QmlFont` | Font family, size, weight, style |
| `QmlPoint` | 2D point (x, y) |
| `QmlRect` | Rectangle (x, y, width, height) |
| `QmlSize` | 2D size (width, height) |
| `Vector2`, `Vector3`, `Vector4` | 2/3/4-component vectors |
| `Matrix4x4` | 4x4 transformation matrix |
| `Quaternion` | Rotation quaternion |

## QmlSharp.Registry

Unified Qt type registry built from three Qt metadata formats.

### Building the Registry

```csharp
RegistryBuilder builder = new();
BuildResult result = builder.BuildOrLoad(new BuildConfig(
    QtDir: qtDir,
    SnapshotPath: "data/qt-registry-snapshots/qt-6.11.0-registry.snapshot.bin",
    ForceRebuild: false,
    ModuleFilter: null,
    IncludeInternal: false));

IRegistryQuery query = result.Query
    ?? throw new InvalidOperationException("Registry build failed.");
```

### Querying

`IRegistryQuery` provides indexed lookups:

```csharp
QmlType? rectangle = query.FindTypeByQmlName("QtQuick", "Rectangle");
QmlType? item = query.FindTypeByQualifiedName("QQuickItem");
IReadOnlyList<QmlType> types = query.GetModuleTypes("QtQuick");
IReadOnlyList<ResolvedProperty> props = query.GetAllProperties("QQuickItem");
IReadOnlyList<QmlType> chain = query.GetInheritanceChain("QQuickRectangle");
```

### Scanning

`QtTypeScanner` discovers Qt type metadata files from a Qt SDK installation, driven by the `QT_DIR` environment variable.

### Key Types

| Type | Description |
|------|-------------|
| `QmlRegistry` | Immutable registry with modules, types, builtins |
| `QmlModule` | Module URI, version, dependencies, type list |
| `QmlType` | Fully resolved type: properties, signals, methods, enums, inheritance |
| `RegistryBuilder` | Registry build/snapshot-load orchestrator |
| `IRegistryQuery` | Query facade with indexed lookups |
| `QtTypeScanner` | Discovers Qt type metadata from the SDK |
| `RegistrySnapshot` | Serializable registry snapshot |

## QmlSharp.Qml.Ast

Immutable QML abstract syntax tree.

### Building AST Nodes

```csharp
QmlDocument doc = new QmlDocumentBuilder()
    .AddModuleImport("QtQuick", "6.11")
    .SetRootObject("Rectangle", root => root
        .Binding("width", Values.Number(100))
        .Binding("color", Values.String("red"))
        .Child("Text", text => text
            .Binding("text", Values.String("Hello"))))
    .Build();
```

### Key Types

| Type | Description |
|------|-------------|
| `QmlDocument` | Root AST node |
| `ObjectDefinitionNode` | QML object with type name and members |
| `BindingNode` | Property binding (name + value) |
| `GroupedBindingNode` | Grouped binding (`font { ... }`) |
| `AttachedBindingNode` | Attached binding (`Layout.fillWidth`) |
| `SignalHandlerNode` | Signal handler (`onClicked: ...`) |
| `PropertyDeclarationNode` | Property declaration |
| `ImportNode` | Import statement |
| `BindingValue` | Abstract binding value (literals, expressions, objects, arrays) |
| `QmlAstValidator` | Structural and semantic validation |

### Traversal and Transforms

| Type | Description |
|------|-------------|
| `QmlAstVisitor` | Visitor pattern for walking AST nodes |
| `QmlAstWalker` | Depth-first tree walker |
| `QmlAstTransformer` | Immutable tree transformation |

## QmlSharp.Qml.Emitter

QML code generation from AST.

### Emitting QML

```csharp
IQmlEmitter emitter = new QmlEmitter();
string qml = emitter.Emit(document, new EmitOptions
{
    IndentSize = 4,
    QuoteStyle = QuoteStyle.Double,
    TrailingNewline = true,
});
```

### Emit with Source Maps

```csharp
EmitResult result = emitter.EmitWithSourceMap(document);
string qml = result.Text;
ISourceMap sourceMap = result.SourceMap;
OutputSpan? span = sourceMap.GetOutputSpan(someNode);
```

### Key Types

| Type | Description |
|------|-------------|
| `IQmlEmitter` / `QmlEmitter` | Converts AST to QML text |
| `EmitOptions` | Formatting options (indent, quotes, semicolons, newlines) |
| `EmitResult` | QML text + source map |
| `ISourceMap` | Read-only source map over emitted positions |
| `EmitPipeline` | Configurable emission pipeline with pre/post processors |

## QmlSharp.Qt.Tools

Wrappers for Qt SDK command-line tools.

### QtToolchain

`QtToolchain` discovers a Qt SDK and resolves tool paths. Individual wrapper classes run each tool:

```csharp
IQtToolchain toolchain = new QtToolchain();
QtInstallation installation = await toolchain.DiscoverAsync(
    new QtToolchainConfig { QtDir = qtDir });

IQmlLint lint = new QmlLint();
QmlLintResult lintResult = await lint.LintFileAsync(qmlFilePath);

IQmlFormat format = new QmlFormat();
QmlFormatResult formatResult = await format.FormatFileAsync(qmlFilePath);
```

### Available Tools

| Wrapper | Qt Tool | Purpose |
|---------|---------|---------|
| `QmlLint` | `qmllint` | Static QML analysis |
| `QmlFormat` | `qmlformat` | QML code formatting |
| `Qmltc` | `qmltc` | QML-to-C++ compilation |
| `QmlDom` | `qmldom` | QML DOM inspection |
| `QmlCachegen` | `qmlcachegen` | AOT QML compilation |
| `QmlImportScanner` | `qmlimportscanner` | Import dependency analysis |
| `QmlTypeRegistrar` | `qmltyperegistrar` | Type registration code generation |
| `Rcc` | `rcc` | Qt resource compilation |
| `QmlRunner` | `qml` | QML runtime execution |
| `QualityGate` | -- | Aggregate pass/fail from multiple tools |

## QmlSharp.Dsl

Fluent C# DSL for declaring QML UIs.

### Key Types

| Type | Description |
|------|-------------|
| `IObjectBuilder` | Builder interface for QML objects |
| `IPropertyCollector` | Collector for grouped/attached bindings |
| `ObjectFactory` | Creates builder instances via DispatchProxy |
| `QmlEnum` | QML enum reference helper |
| `PropertyCollectorFactory` | Creates generated grouped/attached collector proxies |

## QmlSharp.Compiler

C# to QML compiler pipeline.

### Entry Points

```csharp
ICompiler compiler = new QmlCompiler();
CompilationResult result = compiler.Compile(options);
CompilationUnit unit = compiler.CompileFile(filePath, context, options);
OutputResult output = compiler.WriteOutput(result, options);
```

### Key Types

| Type | Description |
|------|-------------|
| `ICompiler` / `QmlCompiler` | Top-level compiler |
| `CompilerOptions` | Compiler configuration |
| `CompilationResult` | Aggregate compilation output |
| `CompilationUnit` | Per-file compilation output |
| `ViewModelSchema` | ViewModel runtime contract |
| `ISourceMapManager` / `SourceMapManager` | Source map creation and lookup |
| `EventBindingsIndex` | Command/effect binding manifest |
| `IEventBindingsBuilder` / `EventBindingsBuilder` | Event binding serialization |
| `IncrementalCompiler` | File-level incremental compilation |
| `CompilerWatcher` | Watch mode with incremental recompilation |

## QmlSharp.Host

Managed runtime host.

### Key Types

| Type | Description |
|------|-------------|
| `QmlSharpEngine` | Primary engine facade |
| `InstanceRegistry` | Active QML instance tracking |
| `CommandRouter` | QML command callback routing |
| `StateSync` | Managed-to-native state synchronization |
| `EffectRouter` | Managed-to-QML effect dispatch |
| `HotReloadCoordinator` | Snapshot/reload/restore orchestration |
| `ArtifactValidator` | Pre-startup artifact validation |
| `ErrorOverlayController` | Development error display |

## QmlSharp.Build

8-stage build pipeline.

### Key Types

| Type | Description |
|------|-------------|
| `IBuildPipeline` / `BuildPipeline` | Full build orchestrator |
| `BuildContext` | Immutable build invocation context |
| `BuildResult` | Aggregate build output |
| `QmlSharpConfig` | Loaded `qmlsharp.json` configuration |
| `IConfigLoader` / `ConfigLoader` | Configuration loading and validation |
| `ICppCodeGenerator` / `CppCodeGenerator` | C++ QObject code generation |
| `ICMakeBuilder` / `CMakeBuilder` | CMake configure/build |
| `IProductLayout` / `ProductLayout` | Output directory assembly |
| `IPlatformDistributor` / `PlatformDistributor` | Platform packaging |
| `Doctor` | Environment diagnostic checks |
| `InitService` | Project scaffolding from templates |
| `CleanService` | Build artifact cleanup |

## QmlSharp.DevTools

Development tools.

### Key Types

| Type | Description |
|------|-------------|
| `IDevServer` / `DevServer` | Dev session facade |
| `FileWatcher` | Source file change monitoring |
| `HotReloadOrchestrator` | Live reload coordination |
| `DevConsole` | Source-map-aware output |
| `IRepl` / `Repl` | Interactive C#/QML REPL |
| `PerfProfiler` | Chrome Trace timing export |
| `ErrorOverlay` | QML error overlay |
| `SchemaDiffer` | Schema change detection |
| `SourceMapLookup` | QML-to-C# location resolution |
| `InstanceMatcher` | Pre/post-reload instance matching |
