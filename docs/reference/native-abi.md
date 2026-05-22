# Native ABI Reference

The QmlSharp native ABI is a flat C function interface exported from the native library (`qmlsharp_native.dll` / `libqmlsharp_native.so` / `libqmlsharp_native.dylib`). The managed host calls these functions through P/Invoke.

This is a **C ABI boundary**, not a C++ object API. All functions use `extern "C"` linkage with `__cdecl` calling convention.

## ABI Version

```c
#define QMLSHARP_ABI_VERSION 1
```

The managed host checks the ABI version before any other call. A mismatch raises `AbiVersionMismatchException`.

## Calling Convention

```c
#if defined(_WIN32)
  #define QMLSHARP_API __declspec(dllexport)  // or dllimport
  #define QMLSHARP_CALL __cdecl
#elif defined(__GNUC__) || defined(__clang__)
  #define QMLSHARP_API __attribute__((visibility("default")))
  #define QMLSHARP_CALL
#endif
```

## String Ownership

- **Caller-owned input**: `const char*` parameters are borrowed. The native side copies if it needs to retain the data.
- **Native-allocated output**: strings returned by `qmlsharp_get_*` functions must be freed by the caller using `qmlsharp_free_string`.

## Function Reference

### Engine Lifecycle

```c
int32_t qmlsharp_get_abi_version(void);
```
Returns the native ABI version. The managed host compares this against its expected version.

```c
void* qmlsharp_engine_init(int32_t argc, const char** argv);
```
Creates `QGuiApplication` and `QQmlEngine`. Returns an opaque engine handle. Returns `NULL` on failure (check `qmlsharp_get_last_error`).

```c
int32_t qmlsharp_engine_exec(void* engine);
```
Enters the Qt event loop. Blocks until the application exits. Returns the exit code.

```c
void qmlsharp_engine_shutdown(void* engine);
```
Destroys the engine and application. Safe to call multiple times.

```c
void qmlsharp_post_to_main_thread(
    qmlsharp_main_thread_callback callback,
    void* user_data);
```
Marshals a callback to the Qt main thread. Used by the managed host to execute native calls from worker threads.

### Type Registration

```c
int32_t qmlsharp_register_type(
    void* engine,
    const char* module_uri,
    int32_t version_major,
    int32_t version_minor,
    const char* type_name,
    const char* schema_id,
    const char* compiler_slot_key,
    qmlsharp_type_registration_callback register_callback);
```
Registers one QML type. `register_callback` is the generated native function that calls Qt's `qmlRegisterType`.

```c
int32_t qmlsharp_register_module(
    void* engine,
    const char* module_uri,
    int32_t version_major,
    int32_t version_minor,
    const qmlsharp_type_registration_entry* entries,
    int32_t entry_count);
```
Batch-registers all types for a module. Each entry carries type name, schema ID, compiler slot key, and registration callback.

**Registration entry struct:**

```c
typedef struct qmlsharp_type_registration_entry {
    const char* type_name;
    const char* schema_id;
    const char* compiler_slot_key;
    qmlsharp_type_registration_callback register_callback;
} qmlsharp_type_registration_entry;
```

### Instance Lifecycle

```c
void qmlsharp_set_instance_callbacks(
    qmlsharp_instance_created_cb on_created,
    qmlsharp_instance_destroyed_cb on_destroyed);
```
Installs callbacks invoked when QML creates or destroys a registered type instance. `on_created` receives instance ID, class name, and compiler slot key. `on_destroyed` receives instance ID.

```c
void qmlsharp_instance_ready(const char* instance_id);
```
Called by the managed host after it has finished initializing the managed ViewModel instance. Opens the native command queue for the instance.

### State Synchronization

These functions push state from the managed side to native QObject properties:

```c
int32_t qmlsharp_sync_state_string(const char* instance_id,
    const char* property_name, const char* value);
int32_t qmlsharp_sync_state_int(const char* instance_id,
    const char* property_name, int32_t value);
int32_t qmlsharp_sync_state_double(const char* instance_id,
    const char* property_name, double value);
int32_t qmlsharp_sync_state_bool(const char* instance_id,
    const char* property_name, int32_t value);
int32_t qmlsharp_sync_state_json(const char* instance_id,
    const char* property_name, const char* json_value);
int32_t qmlsharp_sync_state_batch(const char* instance_id,
    const char* properties_json);
```

All return `0` on success, negative on error.

### Command Dispatch

```c
void qmlsharp_set_command_callback(qmlsharp_command_cb callback);
```

```c
typedef void (*qmlsharp_command_cb)(
    const char* instance_id,
    const char* command_name,
    const char* args_json);
```

The native side calls this callback when a QML signal handler invokes a command. The managed `CommandRouter` dispatches the call to the registered handler.

### Effect Dispatch

```c
int32_t qmlsharp_dispatch_effect(const char* instance_id,
    const char* effect_name, const char* payload_json);
int32_t qmlsharp_broadcast_effect(const char* class_name,
    const char* effect_name, const char* payload_json);
```

Dispatches an effect to one instance or broadcasts to all instances of a class. Returns `0` on success.

### Hot Reload

```c
const char* qmlsharp_capture_snapshot(void* engine);
int32_t qmlsharp_reload_qml(void* engine, const char* qml_source_path);
void qmlsharp_restore_snapshot(void* engine, const char* snapshot_json);
```

Used by the hot reload coordinator. Snapshot captures engine state as JSON, reload loads new QML, and restore rehydrates state. The returned snapshot string must be freed with `qmlsharp_free_string`.

### Error Overlay

```c
void qmlsharp_show_error(void* engine, const char* title,
    const char* message, const char* file_path,
    int32_t line, int32_t column);
void qmlsharp_hide_error(void* engine);
```

Displays or hides a development error overlay in the QML window.

### Diagnostics and Metrics

```c
const char* qmlsharp_get_instance_info(const char* instance_id);
const char* qmlsharp_get_all_instances(void);
const char* qmlsharp_get_metrics(void);
```

Return JSON payloads. Returned strings must be freed with `qmlsharp_free_string`.

### Error Handling

```c
const char* qmlsharp_get_last_error(void);
void qmlsharp_free_string(const char* str);
```

`qmlsharp_get_last_error` returns the most recent error message (thread-local). The returned string must be freed with `qmlsharp_free_string`.

## Return Codes

Most functions return `int32_t`:

| Value | Meaning |
|-------|---------|
| `0` | Success |
| Negative | Error (check `qmlsharp_get_last_error` for details) |
