# QmlSharp

**Build Qt/QML applications in C#.**

## Native Qt Boundary

QmlSharp is a C#-first code-generation toolchain, not a handwritten Qt control
library. All QML-facing Qt classes, properties, signals, slots, and type
registrations must be generated from schema/spec contracts. Framework core must
not add handwritten `QObject`, `QQuickItem`, `QQuickPaintedItem`, `Q_PROPERTY`,
signal/slot, or product-control APIs.

Handwritten native code is allowed for the private Qt engine bootstrap and QRhi
rendering display internals. The rendering backend may use Qt/QRhi classes and
expose stable C ABI, opaque handles, and frame descriptors to managed code.
Other native capability modules, such as FFmpeg, OpenEXR, OpenImageIO, OpenCV,
and similar libraries, expose C ABI contracts and data handles instead of
QML-facing Qt APIs.

### Design TODO: C++ Qt Glue Schema

The current schema work is not complete until it also defines the generated C++
Qt glue: class shape, properties, signals, slots, ownership, type registration,
and ABI routing. Existing QML/runtime schemas are not enough by themselves.
Before adding production native-host controls, design and test this C++ glue
schema/spec from first principles.

The prior QML + Rust reference is useful for product behavior and workflow
comparison, but it is not a valid source of truth for generated C++ Qt classes
or the QRhi host architecture. Do not fill this gap with handwritten
`QObject`/`QQuickItem` controls in framework core.

QmlSharp is a managed framework and toolchain for Qt 6. Write views, ViewModels,
and application logic in C# through a fluent DSL. The compiler generates QML,
schema contracts, generated native Qt bindings, source maps, and build outputs.
A private Qt host loads and runs the generated application.

## At a glance

```csharp
// A ViewModel with state, commands, and effects
[ViewModel]
public partial class CounterViewModel
{
    [State]   public int  Count { get; set; }
    [Command] public void Increment() => Count++;
    [Effect]  public event Action<string>? ShowToast;
}
```

```csharp
// A view declared in fluent C#
public static View Counter(CounterViewModel vm) =>
    Column(
        Text(vm.State(s => s.Count)).FontSize(24),
        Button("Increment").OnClicked(vm.Increment)
    );
```

> Examples are conceptual. The public API is still evolving.

## How it works

1. **C# is the source of truth.** Views use a fluent DSL; ViewModels use
   `[State]`, `[Command]`, and `[Effect]` attributes.
2. **The compiler generates everything else.** QML files, `.schema.json`
   contracts, QML-facing C++ Qt glue, module metadata, and source maps.
3. **Interop is flat C ABI + P/Invoke.** No COM, no C++/CLI, no Node.js.
4. **Qt 6.11 is the runtime baseline.** Discovered through `QT_DIR`.

## Status

QmlSharp is in early development. The compiler, runtime contracts, and public
APIs are stabilizing and may change between releases.

## Documentation

See the [full documentation](docs/index.md) for architecture, API reference,
configuration, and contributor guides.

## License

[MIT](LICENSE)
