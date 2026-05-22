# QML Generation

QmlSharp transforms fluent C# DSL declarations into QML files through a multi-stage compiler pipeline. This page describes how C# source becomes generated QML output.

## Compiler Pipeline Stages

The compiler processes each source file through these stages:

### 1. C# Analysis (`CSharpAnalyzer`)

Uses Roslyn (`Microsoft.CodeAnalysis.CSharp`) to:
- Parse C# source into a semantic model
- Discover View classes (classes that use the fluent DSL)
- Discover ViewModel classes (annotated with `[ViewModel]`)
- Analyze import dependencies between files
- Report syntax diagnostics

### 2. ViewModel Extraction (`ViewModelExtractor`)

For each discovered ViewModel:
- Extracts `[State]` properties with types and defaults
- Extracts `[Command]` methods with parameters
- Extracts `[Effect]` events with payload types
- Detects lifecycle methods (`OnMounted`, `OnUnmounting`, `OnHotReload`)
- Allocates stable IDs via `IdAllocator`
- Produces a `ViewModelSchema`

### 3. DSL Transformation (`DslTransformer`)

For each discovered View:
- Identifies factory method calls (e.g., `Column()`, `Text()`)
- Walks fluent method chains (`.SetProperty()`, `.SetBinding()`, `.HandleSignal()`)
- Resolves `vm.State(...)` and `vm.Command(...)` references
- Recognizes grouped bindings, attached bindings, and child composition
- Produces QML AST nodes (`ObjectDefinitionNode`, `BindingNode`, etc.)

### 4. Post-Processing (`PostProcessor`)

Injects runtime-required elements into the AST:
- **Import injection** -- adds necessary `import` statements (QtQuick, application module, etc.)
- **Connection injection** -- wires effect listeners as QML `Connections` objects
- **Lifecycle injection** -- adds `Component.onCompleted` and `Component.onDestruction` handlers
- Validates:
  - **P001**: duplicate QML `id` detection
  - **P003**: command parameter arity checks
  - **P004**: state binding target validation

### 5. QML Emission (`QmlEmitter`)

Converts the post-processed QML AST into formatted QML text:
- Respects configurable emit options (indentation, quotes, semicolons, newlines)
- Produces source maps linking QML output positions back to C# source positions
- Deterministic output (sorted imports, normalized member ordering)

### 6. Output Writing (`CompilerOutputWriter`)

Writes all compilation artifacts to disk:
- `.qml` files
- `.schema.json` files (one per ViewModel)
- `event-bindings.json` (aggregate command/effect binding manifest)
- Source map files

## QML AST

The intermediate QML representation is a tree of immutable record types rooted at `QmlDocument`:

```
QmlDocument
  Pragmas ........ PragmaNode[]
  Imports ........ ImportNode[]
  RootObject ..... ObjectDefinitionNode
                     TypeName
                     Members:
                       IdAssignmentNode
                       BindingNode
                       GroupedBindingNode
                       AttachedBindingNode
                       ArrayBindingNode
                       PropertyDeclarationNode
                       SignalDeclarationNode
                       SignalHandlerNode
                       FunctionDeclarationNode
                       EnumDeclarationNode
                       InlineComponentNode
                       BehaviorOnNode
                       ObjectDefinitionNode (children)
```

The AST supports the full Qt 6.11 QML syntax, including pragmas (`Singleton`, `ComponentBehavior`, etc.), all import forms (module, directory, JavaScript), and all binding value types (literals, expressions, script blocks, object values, arrays).

## Import Resolution

The `ImportResolver` manages QML import statements:
- Deduplicates imports across multiple transformation stages
- Merges version-compatible imports
- Injects framework imports when AST nodes reference Qt types
- Preserves user-specified qualifiers

## Compilation Units

Each View file produces a `CompilationUnit`:

```
CompilationUnit
  SourceFilePath ..... original C# file
  ViewClassName ...... C# view class
  ViewModelClassName . bound ViewModel class
  QmlText ............ generated QML string
  Schema ............. ViewModelSchema
  Document ........... QmlDocument AST
  SourceMap .......... C#-to-QML position mappings
  Diagnostics ........ per-file diagnostics
  Stats .............. elapsed time, byte count
```

Units are aggregated into a `CompilationResult` with project-level diagnostics, statistics, and an `EventBindingsIndex`.

## Incremental Compilation

The `IncrementalCompiler` caches compilation results per file and only recompiles files whose content has changed. The incremental strategy operates at the `CompilationUnit` level -- a changed file re-enters the full pipeline, but unchanged files reuse their cached output.

## Watch Mode

The `CompilerWatcher` monitors source files for changes and triggers incremental compilation. It integrates with the dev server for live hot-reload workflows.
