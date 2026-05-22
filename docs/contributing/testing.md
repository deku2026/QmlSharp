# Testing

QmlSharp uses xUnit for .NET tests and Google Test for native tests. Every module has a corresponding test project.

## Test Projects

| Project | Module Under Test |
|---------|------------------|
| `QmlSharp.Registry.Tests` | Qt type registry |
| `QmlSharp.Qml.Ast.Tests` | QML AST |
| `QmlSharp.Qml.Emitter.Tests` | QML emitter |
| `QmlSharp.Qt.Tools.Tests` | Qt toolchain wrappers |
| `QmlSharp.Dsl.Tests` | Fluent DSL |
| `QmlSharp.Dsl.Generator.Tests` | DSL code generator |
| `QmlSharp.Compiler.Tests` | Compiler pipeline |
| `QmlSharp.Host.Tests` | Runtime host |
| `QmlSharp.Build.Tests` | Build pipeline |
| `QmlSharp.DevTools.Tests` | Dev tools |
| `QmlSharp.Integration.Tests` | Cross-module integration |

## Test Categories

Tests are organized by concern:

- **Contracts** -- verify public API surface, record shapes, enum values
- **Closure** -- verify module boundary coverage (all public types tested)
- **Parity** -- verify consistency between related components
- **Infrastructure** -- verify test harness and helper utilities
- **Unit** -- per-class behavior tests
- **Integration** -- multi-component flows
- **Performance** -- throughput and regression benchmarks
- **E2E** -- end-to-end golden-file tests

## Qt-Dependent Tests

Some tests require a Qt SDK installation. These use custom xUnit attributes:

- `[SkipUnlessEnvironmentVariableFact("QT_DIR")]` -- skips if `QT_DIR` is not set
- `[RequiresQtFact]` -- skips if Qt tools are not available

Qt-dependent tests are in their own collection to avoid parallel execution conflicts.

## Golden File Tests

The emitter and compiler include golden-file tests that compare generated output against known-good reference files. Golden files live alongside the test projects.

To update golden files after an intentional output change, run the tests with the update flag or manually replace the expected files.

## Fixture Projects

The `tests/fixtures/projects/` directory contains small QmlSharp projects used by integration tests:

| Fixture | Purpose |
|---------|---------|
| `counter-application` | Basic counter app |
| `consuming-application` | App consuming a library |
| `qmlsharp-library` | Reusable library |
| `assets-application` | Asset bundling |
| `compilation-error` | Intentional errors for diagnostic testing |
| `invalid-config` | Invalid configuration for validation testing |
| `native-prebuilt-application` | Pre-built native library |
| `dev-tools-counter-flow` | Dev tools integration |

## Running Tests

Run all tests:

```bash
dotnet test QmlSharp.slnx --configuration Debug
```

Run one module's tests:

```bash
dotnet test tests/QmlSharp.Compiler.Tests
```

Run tests matching a filter:

```bash
dotnet test QmlSharp.slnx --filter "FullyQualifiedName~PostProcessor"
```

## Native Tests

Native tests use Google Test and are built alongside the native library:

```bash
./build/debug/bin/qmlsharp_native_tests
./build/debug/bin/qmlsharp_native_performance_tests
```

The native test fixtures include mock ViewModels (`RegistrationCounterViewModel`, `RegistrationStatusViewModel`) for integration and smoke testing.
