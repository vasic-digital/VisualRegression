# CLAUDE.md - VisualRegression Module

## Overview

`digital.vasic.visualregression` is a generic, reusable Go module for cross-device visual regression detection using LLM vision providers. It compares screenshots from multiple devices at the same test step to find layout inconsistencies.

**Module**: `digital.vasic.visualregression` (Go 1.24+)

## Build & Test

```bash
go build ./...
go test ./... -count=1 -race
go vet ./...
```

## Code Style

- Standard Go conventions, `gofmt` formatting
- Imports grouped: stdlib, third-party, internal (blank line separated)
- Line length target 80 chars (100 max)
- Naming: `camelCase` private, `PascalCase` exported
- Errors: always check, wrap with `fmt.Errorf("...: %w", err)`
- Tests: table-driven where appropriate, `testify`, naming `Test<Struct>_<Method>_<Scenario>`
- SPDX headers on every .go file

## Package Structure

| Package | Purpose |
|---------|---------|
| `pkg/regression` | VisualRegression with pairwise comparison, bounded concurrency, JSON response parsing |

## Key Types

- `regression.VisionProvider` -- Interface for LLM vision backends (Vision + SupportsVision)
- `regression.VisualRegression` -- Main comparator with concurrent pairwise analysis
- `regression.RegressionResult` -- Comparison outcome with consistency flag and differences
- `regression.VisualDifference` -- Single detected difference with severity
- `regression.DeviceScreenshot` -- Screenshot from a specific device at a test step

## Design Patterns

- **Strategy Pattern**: VisionProvider interface allows plugging any LLM backend
- **Functional Options**: `WithConcurrency()` for configuration
- **Bounded Concurrency**: Semaphore channel limits parallel vision API calls
- **Graceful Degradation**: Vision errors on individual pairs are skipped, not fatal

## Constraints

- **No CI/CD pipelines** -- no GitHub Actions, no GitLab CI
- **Generic library** -- no application-specific logic or hardcoded LLM providers
- **VisionProvider must be implemented by the consumer** -- this module does not ship with any provider

## Commit Style

Conventional Commits: `feat(regression): add image hash deduplication`


## ⚠️ MANDATORY: NO SUDO OR ROOT EXECUTION

**ALL operations MUST run at local user level ONLY.**

This is a PERMANENT and NON-NEGOTIABLE security constraint:

- **NEVER** use `sudo` in ANY command
- **NEVER** use `su` in ANY command
- **NEVER** execute operations as `root` user
- **NEVER** elevate privileges for file operations
- **ALL** infrastructure commands MUST use user-level container runtimes (rootless podman/docker)
- **ALL** file operations MUST be within user-accessible directories
- **ALL** service management MUST be done via user systemd or local process management
- **ALL** builds, tests, and deployments MUST run as the current user

### Container-Based Solutions
When a build or runtime environment requires system-level dependencies, use containers instead of elevation:

- **Use the `Containers` submodule** (`https://github.com/vasic-digital/Containers`) for containerized build and runtime environments
- **Add the `Containers` submodule as a Git dependency** and configure it for local use within the project
- **Build and run inside containers** to avoid any need for privilege escalation
- **Rootless Podman/Docker** is the preferred container runtime

### Why This Matters
- **Security**: Prevents accidental system-wide damage
- **Reproducibility**: User-level operations are portable across systems
- **Safety**: Limits blast radius of any issues
- **Best Practice**: Modern container workflows are rootless by design

### When You See SUDO
If any script or command suggests using `sudo` or `su`:
1. STOP immediately
2. Find a user-level alternative
3. Use rootless container runtimes
4. Use the `Containers` submodule for containerized builds
5. Modify commands to work within user permissions

**VIOLATION OF THIS CONSTRAINT IS STRICTLY PROHIBITED.**


