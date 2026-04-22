# AGENTS.md - Multi-Agent Coordination Guide

## Overview

This document provides guidance for AI agents working with the `digital.vasic.visualregression` module.

## Module Identity

- **Module path**: `digital.vasic.visualregression`
- **Language**: Go 1.24+
- **Dependencies**: `github.com/stretchr/testify` (tests only)
- **Scope**: Generic, reusable visual regression detection. No application-specific logic.

## Package Responsibilities

| Package | Owner Concern | Agent Must Not |
|---------|--------------|----------------|
| `pkg/regression` | Pairwise comparison, JSON parsing, severity classification | Add hardcoded LLM providers, import non-stdlib production deps |

## Coordination Rules

### 1. Interface Contracts

The `VisionProvider` interface is a stability boundary. Breaking changes require explicit human approval:

- `Vision(ctx, image, prompt) (*VisionResponse, error)`
- `SupportsVision() bool`

### 2. JSON Response Contract

The `comparisonPrompt` template and `comparisonJSON` struct define the LLM interaction contract. Changes to the expected JSON shape must be backwards-compatible.

### 3. Test Requirements

- All tests use `testify/assert` and `testify/require`.
- Test naming convention: `Test<Struct>_<Method>_<Scenario>`.
- Mock providers use `atomic.Int32` for thread-safe call counting.
- Race detector must pass: `go test ./... -race`.

## Agent Workflow

### Before Making Changes

```bash
go build ./...
go test ./... -count=1 -race
```

### After Making Changes

```bash
gofmt -w .
go vet ./...
go test ./... -count=1 -race
```

### Commit Convention

```
<type>(<package>): <description>

# Examples:
feat(regression): add image hash deduplication
fix(regression): handle empty JSON response from provider
test(regression): add five-device pairwise coverage
```

## Boundaries

### What Agents May Do

- Fix bugs in any package.
- Add tests for uncovered code paths.
- Refactor internals without changing exported APIs.
- Add new exported methods that extend existing types.
- Update documentation to match code.

### What Agents Must Not Do

- Break existing exported interfaces or method signatures.
- Add application-specific logic (this is a generic library).
- Ship concrete LLM provider implementations.
- Introduce new external dependencies without human approval.
- Modify `go.mod` without explicit instruction.

## Key Files

| File | Purpose |
|------|---------|
| `pkg/regression/visual.go` | All production code |
| `pkg/regression/visual_test.go` | All tests |
| `go.mod` | Module definition |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | Agent build/test guidance |


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


