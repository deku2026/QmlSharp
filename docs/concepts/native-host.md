# Native Host

QmlSharp's native layer is a thin C/C++ shell around Qt's `QQmlEngine` and `QGuiApplication`. It provides the minimum runtime surface needed to load generated QML, register managed ViewModel types, and bridge state/command/effect traffic across the managed/native boundary.

## Architecture

```
Managed (.NET)                          Native (C/C++)
+--------------------+                  +------------------------+
| QmlSharpEngine     |   P/Invoke       | qmlsharp_abi.h         |
|   Initialize()     | <--------------> |   qmlsharp_engine_init |
|   RegisterTypes()  |   flat C ABI     |   qmlsharp_register_*  |
|   Run()            |                  |   qmlsharp_sync_state_*|
|   SyncState()      |                  |   qmlsharp_dispatch_*  |
|   DispatchEffect() |                  |   qmlsharp_reload_qml  |
+--------------------+                  +------------------------+
                                        | QQmlEngine             |
                                        | QGuiApplication        |
                                        | Generated QObject types|
                                        +------------------------+
```

## C ABI Boundary

All interop uses `extern "C"` functions with `__cdecl` calling convention. The ABI header (`native/include/qmlsharp/qmlsharp_abi.h`) exports a flat function set grouped by capability:

### Engine Lifecycle

| Function | Purpose |
|----------|---------|
| `qmlsharp_get_abi_version` | Returns the ABI version for compatibility checks |
| `qmlsharp_engine_init` | Creates `QGuiApplication` and `QQmlEngine`, returns opaque handle |
| `qmlsharp_engine_exec` | Enters the Qt event loop |
| `qmlsharp_engine_shutdown` | Destroys engine and application |
| `qmlsharp_post_to_main_thread` | Marshals a callback to the Qt main thread |

### Type Registration

| Function | Purpose |
|----------|---------|
| `qmlsharp_register_type` | Registers one QML type with schema and slot key |
| `qmlsharp_register_module` | Batch-registers all types for a QML module |

### Instance Lifecycle

| Function | Purpose |
|----------|---------|
| `qmlsharp_set_instance_callbacks` | Installs created/destroyed callbacks |
| `qmlsharp_instance_ready` | Signals that managed initialization is complete |

### State Synchronization

| Function | Purpose |
|----------|---------|
| `qmlsharp_sync_state_string` | Pushes a string property |
| `qmlsharp_sync_state_int` | Pushes an int32 property |
| `qmlsharp_sync_state_double` | Pushes a double property |
| `qmlsharp_sync_state_bool` | Pushes a boolean property |
| `qmlsharp_sync_state_json` | Pushes a JSON-serialized property |
| `qmlsharp_sync_state_batch` | Pushes multiple properties atomically |

### Command Dispatch

| Function | Purpose |
|----------|---------|
| `qmlsharp_set_command_callback` | Installs the managed command handler |

When a QML signal fires, the native side calls the registered command callback with the instance ID, command name, and JSON arguments.

### Effects

| Function | Purpose |
|----------|---------|
| `qmlsharp_dispatch_effect` | Sends an effect to one instance |
| `qmlsharp_broadcast_effect` | Sends an effect to all instances of a class |

### Hot Reload

| Function | Purpose |
|----------|---------|
| `qmlsharp_capture_snapshot` | Captures engine state as JSON |
| `qmlsharp_reload_qml` | Reloads a QML source file |
| `qmlsharp_restore_snapshot` | Restores engine state from JSON |

### Error Overlay

| Function | Purpose |
|----------|---------|
| `qmlsharp_show_error` | Displays a development error overlay |
| `qmlsharp_hide_error` | Hides the error overlay |

### Diagnostics

| Function | Purpose |
|----------|---------|
| `qmlsharp_get_instance_info` | Returns JSON for one instance |
| `qmlsharp_get_all_instances` | Returns JSON for all instances |
| `qmlsharp_get_metrics` | Returns runtime metrics JSON |
| `qmlsharp_get_last_error` | Returns the last error string |
| `qmlsharp_free_string` | Frees a string allocated by the native side |

## String Ownership

Strings passed **into** the native side (`const char*` parameters) are borrowed -- the native side copies them if needed. Strings **returned** by the native side must be freed by the caller using `qmlsharp_free_string`.

## ABI Versioning

The managed host checks `qmlsharp_get_abi_version()` before any other call. If the returned version does not match `NativeHostAbi.SupportedAbiVersion`, initialization fails with `AbiVersionMismatchException`. The current ABI version is `1`.

## QmlSharpEngine (Managed Side)

`QmlSharpEngine` is the primary managed entry point. It wraps the native ABI behind a high-level C# API:

```csharp
using QmlSharpEngine engine = new("path/to/qmlsharp_native.dll");
engine.Initialize(distDirectory, args, rootQmlPath);
engine.RegisterTypes(schemas);
return engine.Run(rootQmlPath);
```

The engine manages:
- **InstanceRegistry** -- tracks active QML object instances
- **CommandRouter** -- routes QML command callbacks to managed handlers
- **StateSync** -- pushes managed state to native QObject properties
- **EffectRouter** -- dispatches effects from managed code to QML
- **HotReloadCoordinator** -- orchestrates snapshot/reload/restore cycles
- **ErrorOverlayController** -- shows development diagnostics in the QML UI
- **ArtifactValidator** -- validates build outputs before engine initialization

## Build Configuration

The native library is built with CMake. The `QmlSharp.Build` module generates:
- C++ QObject subclass headers and implementations from `ViewModelSchema`
- `type_registration.cpp` with `qmlsharp_register_module` calls
- `CMakeLists.txt` linking against Qt6::Quick and Qt6::Qml

The `CppGenerationOptions.AbiSourceDir` points to the handwritten ABI sources under `native/src/`.
