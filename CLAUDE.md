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
