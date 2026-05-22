# ViewModels

QmlSharp uses a ViewModel pattern where application state, user commands, and side effects are declared through C# attributes. The compiler extracts these declarations and generates the runtime contracts that connect the managed ViewModel to its QML representation.

## Declaring a ViewModel

A ViewModel is a C# class annotated with `[ViewModel]`. Within it, three attributes define the contract:

```csharp
[ViewModel]
public partial class CounterViewModel
{
    [State]   public int  Count { get; set; }
    [Command] public void Increment() => Count++;
    [Effect]  public event Action<string>? ShowToast;
}
```

### `[State]` -- Observable Properties

Properties marked `[State]` become observable QML properties. The runtime synchronizes their values from the managed side to the native QObject instance using typed state-sync calls (`qmlsharp_sync_state_int`, `qmlsharp_sync_state_string`, etc.).

Each state property is assigned a stable member ID by the compiler's `IdAllocator` (FNV-1a hash with collision handling).

State properties support primitive values (`bool`, integer types, floating point types, `string`), JSON values, and supported list shapes. The extractor records default values where they are known and preserves the original C# member name alongside the QML-facing camelCase name.

### `[Command]` -- User Actions

Methods marked `[Command]` are callable from QML signal handlers. When a QML signal fires (e.g., `onClicked`), the native host dispatches the command to the managed `CommandRouter`, which invokes the registered handler.

Commands support:
- Synchronous and asynchronous (`async`) methods
- Typed parameters (serialized as JSON across the boundary)
- Stable command IDs for deterministic binding

### `[Effect]` -- Side Effects

Events marked `[Effect]` are dispatched from the managed side to QML when state changes require visual feedback beyond property binding. Effects can target a single instance or broadcast to all instances of a ViewModel class.

Effects support:
- `Action` and `Action<T>` event handlers
- Zero or one typed payload value
- Per-instance dispatch (`DispatchEffect`)
- Class-wide broadcast (`BroadcastEffect`)

## ViewModelSchema

The compiler extracts all `[State]`, `[Command]`, and `[Effect]` declarations into a `ViewModelSchema` record:

```
ViewModelSchema
  ClassName .......... C# class name
  ModuleName ......... QML module name
  ModuleUri .......... QML module URI
  ModuleVersion ...... major.minor
  CompilerSlotKey .... deterministic hash for native type registration
  Properties ......... StateEntry[] (name, type, default, memberId)
  Commands ........... CommandEntry[] (name, parameters, commandId)
  Effects ............ EffectEntry[] (name, payloadType, effectId)
  Lifecycle .......... onMounted, onUnmounting, hotReload flags
```

This schema is the single contract between the compiler and all downstream consumers: the emitter, the C++ code generator, the native type registrar, and the runtime host.

## Lifecycle

ViewModels can implement lifecycle callbacks detected by the compiler:

| Callback | When |
|----------|------|
| `OnMounted` | After the QML component is fully loaded and the instance is ready |
| `OnUnmounting` | Before the QML component is destroyed |
| `OnHotReload` | During a hot reload cycle, after state has been rehydrated |

## ID Allocation

The compiler assigns stable integer IDs to state properties, commands, and effects using FNV-1a hashing over the ViewModel/member key with separate namespaces for state, command, and effect entries. Collisions are resolved by deterministic linear probing. These IDs are recorded in schemas and `event-bindings.json`.
