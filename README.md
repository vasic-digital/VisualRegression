# digital.vasic.visualregression

Cross-device visual regression detection using LLM vision providers. Compares screenshots taken at the same test step across multiple devices to detect layout inconsistencies, missing elements, and platform-specific rendering bugs.

## Installation

```bash
go get digital.vasic.visualregression
```

## Quick Start

```go
package main

import (
    "context"
    "fmt"

    "digital.vasic.visualregression/pkg/regression"
)

func main() {
    // Implement the VisionProvider interface with your
    // preferred LLM vision backend.
    provider := NewMyVisionProvider()

    // Create a regression detector with concurrent comparisons.
    vr := regression.NewVisualRegression(
        provider,
        regression.WithConcurrency(3),
    )

    // Compare screenshots from multiple devices at the same step.
    result, err := vr.Compare(context.Background(),
        []regression.DeviceScreenshot{
            {
                Device:     "phone-1",
                Platform:   "android",
                Screenshot: phoneScreenshot,
                Step:       1,
                StepName:   "login screen",
            },
            {
                Device:     "tablet-1",
                Platform:   "android",
                Screenshot: tabletScreenshot,
                Step:       1,
                StepName:   "login screen",
            },
            {
                Device:     "tv-1",
                Platform:   "androidtv",
                Screenshot: tvScreenshot,
                Step:       1,
                StepName:   "login screen",
            },
        },
    )
    if err != nil {
        panic(err)
    }

    fmt.Printf("Consistent: %v, Differences: %d\n",
        result.Consistent, len(result.Differences))

    for _, diff := range result.Differences {
        fmt.Printf("[%s] %s vs %s: %s\n",
            diff.Severity, diff.DeviceA, diff.DeviceB,
            diff.Description)
    }
}
```

## VisionProvider Interface

You must implement this interface to use VisualRegression:

```go
type VisionProvider interface {
    Vision(ctx context.Context, image []byte, prompt string) (*VisionResponse, error)
    SupportsVision() bool
}
```

Any LLM with vision capabilities can back this interface (OpenAI GPT-4V, Google Gemini, Ollama with vision models, etc.).

## API Reference

### VisualRegression

| Method | Description |
|--------|-------------|
| `NewVisualRegression(provider, ...Option)` | Create a detector with a vision provider |
| `Compare(ctx, screenshots) (*RegressionResult, error)` | Compare screenshots at one test step |
| `CompareMultipleSteps(ctx, steps) ([]*RegressionResult, error)` | Compare across multiple steps |

### Options

| Option | Description |
|--------|-------------|
| `WithConcurrency(n int)` | Max concurrent pairwise comparisons (default 1) |

### RegressionResult

| Field | Type | Description |
|-------|------|-------------|
| `Step` | `int` | Test step number |
| `StepName` | `string` | Optional step description |
| `Devices` | `[]string` | All compared devices |
| `Differences` | `[]VisualDifference` | Detected visual differences |
| `Consistent` | `bool` | True when no differences found |
| `ComparisonsMade` | `int` | Number of pairwise comparisons |
| `CriticalCount()` | `int` | Count of critical-severity differences |

### Severity Levels

| Constant | Description |
|----------|-------------|
| `SeverityCritical` | Blocking visual difference |
| `SeverityWarning` | Noticeable visual difference |
| `SeverityInfo` | Minor visual difference |

### Utility Functions

| Function | Description |
|----------|-------------|
| `ConsistencyRate(results) float64` | Fraction of consistent steps (0-1) |
| `TotalDifferences(results) int` | Total differences across all steps |
| `ValidSeverity(s string) bool` | Check if a severity string is valid |

## License

Apache-2.0
