# Diagnostics Reference

QmlSharp produces structured diagnostics at the compiler and build levels. Each diagnostic has a stable code, severity, message, and optional source location.

## Severity Levels

| Level | Meaning |
|-------|---------|
| `Info` | Informational message, does not affect build success |
| `Warning` | Potential issue that does not block compilation |
| `Error` | Compilation or build failure |
| `Fatal` | Unrecoverable failure |

A compilation or build is successful when it has no `Error` or `Fatal` diagnostics.

## Compiler Diagnostics

Compiler diagnostics are reported by the `QmlSharp.Compiler` pipeline. Codes are stable constants on `DiagnosticCodes`.

### Attribute and Contract Diagnostics

| Code | Severity | Description |
|------|----------|-------------|
| `QMLSHARP-A001` | Error | Invalid `[State]` usage |
| `QMLSHARP-A002` | Error | Invalid `[Command]` usage |
| `QMLSHARP-A003` | Error | Invalid `[Effect]` usage |
| `QMLSHARP-A005` - `QMLSHARP-A007` | Error | Duplicate state, command, or effect names |
| `QMLSHARP-A008` - `QMLSHARP-A010` | Error | Unsupported state/command/effect shapes |

### DSL Transform Diagnostics

| Prefix | Description |
|--------|-------------|
| `QMLSHARP-T` | QML type lookup, property binding, signal, attached type, and DSL call-chain diagnostics |
| `QMLSHARP-P` | Post-processing diagnostics for generated imports, ViewModel injection, binding targets, command targets, effects, and slot-key collisions |

### Emission and Pipeline Diagnostics

| Prefix | Description |
|--------|-------------|
| `QMLSHARP-C` | QML emission, output writing, schema serialization, source-map writing, and optional Qt validation |
| `QMLSHARP-G` | Internal compiler, Roslyn compilation, and project loading failures |

## Build Diagnostics

Build diagnostics are reported by the `QmlSharp.Build` pipeline.

### Configuration Diagnostics

| Severity | Description |
|----------|-------------|
| `Warning` | Non-critical configuration issue (e.g., deprecated field) |
| `Error` | Invalid configuration that prevents the build from starting |

Configuration errors produce a `ConfigParseException` with one or more `BuildDiagnostic` records.

### Build Stage Diagnostics

Each build stage can report diagnostics through `BuildDiagnostic`:

```csharp
public sealed record BuildDiagnostic(
    string Code,
    BuildDiagnosticSeverity Severity,
    string Message,
    BuildPhase? Phase,
    string? FilePath);
```

### Artifact Validation Diagnostics

Pre-startup artifact validation (`ArtifactValidator`) reports:

| Code | Description |
|------|-------------|
| `AV-001` | Missing or invalid startup manifest |
| `AV-002` | Missing native library |
| `AV-003` | Native ABI version mismatch |
| `AV-004` | Missing or invalid schema artifact |
| `AV-005` | Missing or invalid optional `event-bindings.json` |
| `AV-006` | Event binding does not match schema command metadata |
| `AV-007` | Native schema registration failed |

## Diagnostic Locations

Compiler diagnostics include source locations when available:

```csharp
public sealed record SourceLocation(
    string? FilePath,
    int? Line,
    int? Column);
```

Source-map-aware diagnostics in the dev console show both the generated QML location and the original C# source location.

## Diagnostic Output

- **CLI**: diagnostics are printed to stderr with file:line:column format
- **JSON mode** (`--json`): diagnostics are included in the structured output
- **Dev console**: diagnostics are formatted with source map lookups
- **Error overlay**: diagnostics are displayed in the QML application window
