# Fluent DSL

QmlSharp provides a fluent C# DSL for declaring QML user interfaces. Instead of hand-writing QML files, you describe your UI in C# using builder methods that mirror QML's declarative structure. The compiler transforms these declarations into QML at build time.

## Core Concepts

The DSL is built on two interfaces:

- **`IObjectBuilder`** -- Represents a QML object (like `Rectangle`, `Text`, or `Button`). Provides methods for setting properties, adding bindings, attaching signal handlers, and composing child objects.
- **`IPropertyCollector`** -- Collects property entries within grouped or attached binding callbacks.

Objects are created through `ObjectFactory`, which uses `DispatchProxy` to generate runtime implementations of generated builder interfaces.

## Writing a View

A view is a C# class that declares its UI tree using builder methods:

```csharp
// Conceptual -- the exact fluent API surface is still evolving.
public static View Counter(CounterViewModel vm) =>
    Column(
        Text(vm.State(s => s.Count)).FontSize(24),
        Button("Increment").OnClicked(vm.Increment)
    );
```

The compiler discovers these view declarations, walks the fluent call chain, and emits the corresponding QML AST.

## Builder API

`IObjectBuilder` supports the following operations:

| Method | Purpose |
|--------|---------|
| `Id(string id)` | Sets the QML `id` for this object |
| `Child(IObjectBuilder child)` | Adds one child object |
| `Children(params IObjectBuilder[] children)` | Adds multiple child objects |
| `SetProperty(string name, object? value)` | Sets a literal property value |
| `SetBinding(string name, string expression)` | Sets a QML expression binding |
| `AddGrouped(string group, Action<IPropertyCollector> configure)` | Adds grouped bindings (e.g. `font { ... }`) |
| `AddAttached(string type, Action<IPropertyCollector> configure)` | Adds attached bindings (e.g. `Layout.fillWidth`) |
| `HandleSignal(string handler, string body)` | Adds a signal handler |
| `Build()` | Produces an immutable `ObjectDefinitionNode` |

## Generated Type-Safe Wrappers

The `QmlSharp.Dsl.Generator` module reads the Qt type registry and generates type-safe builder interfaces for Qt types. Instead of writing `SetProperty("text", "Hello")`, generated wrappers provide strongly-typed methods like `.Text("Hello")`.

The generator produces:

- Property setter methods (`.Text(string)`, `.Width(double)`)
- Signal handler methods (`.OnClicked(Action)`)
- Enum accessors
- Grouped and attached property helpers
- Inheritance-aware method chains

## Property Collector

For grouped and attached properties, the DSL uses `IPropertyCollector`:

```csharp
builder.AddGrouped("font", font =>
{
    font.SetProperty("pixelSize", 16);
    font.SetProperty("bold", true);
});
```

This emits the QML grouped binding:

```qml
font {
    pixelSize: 16
    bold: true
}
```

## Name Conventions

Generated wrappers and the compiler map between C# naming conventions (PascalCase methods and properties) and QML naming conventions (camelCase properties, `on<Signal>` handlers).

## QML Value Conversion

The DSL runtime maps common .NET and QmlSharp value types to QML-compatible literal values. For example, `QmlColor` maps to a QML color string, vector and geometry values map to Qt factory expressions, and enum tokens map to QML enum references.
