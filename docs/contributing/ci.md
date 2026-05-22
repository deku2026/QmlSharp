# Continuous Integration

QmlSharp runs CI on three platforms through GitHub Actions. All workflows trigger on pushes to `main` and on pull requests targeting `main`.

## Workflows

| Workflow | File | Platform |
|----------|------|----------|
| Windows CI | `.github/workflows/windows-ci.yml` | `windows-2025-vs2026` |
| Linux CI | `.github/workflows/linux-ci.yml` | Ubuntu |
| macOS CI | `.github/workflows/macos-ci.yml` | macOS |
| CodeQL | `.github/workflows/codeql.yml` | Security analysis |

## Windows CI Pipeline

The Windows workflow is the most comprehensive:

1. **Checkout** repository
2. **Setup .NET SDK** from `global.json`
3. **Restore Qt 6.11.0** from cache (or install from download.qt.io)
4. **Export Qt environment** (`QT_DIR`, `Qt6_DIR`, PATH)
5. **Setup MSVC** developer environment (x64)
6. **Detect Windows SDK** (for `rc.exe`)
7. **Verify tool versions**
8. **Restore** Windows solution (`QmlSharp.Win.slnx`)
9. **Build** Windows solution (MSBuild, Debug/x64, warnings as errors)
10. **Test native targets** (smoke tests and performance tests)
11. **Test .NET solution** (`dotnet test`, single-threaded)
12. **Check formatting** (`dotnet format --verify-no-changes`)

## Qt Installation in CI

Qt 6.11.0 is installed by downloading `qtbase` and `qtdeclarative` archives directly from `download.qt.io`. Archives are SHA256-verified before extraction. The installation is cached across runs.

## Concurrency

Workflows use concurrency groups per branch to avoid redundant runs:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false
```

## Permissions

All workflows run with read-only `contents` permission.

## Dependabot

Dependabot is configured (`.github/dependabot.yml`) to monitor NuGet package dependencies.

## Pre-Commit

A `.pre-commit-config.yaml` is present for local pre-commit hook configuration.

## Branch Protection

The `main` branch is protected. All changes must go through a pull request. Direct pushes to `main` are not permitted.
