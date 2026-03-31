# Architecture -- VisualRegression

## Purpose

Cross-device visual regression detection using LLM vision providers. Compares screenshots taken at the same test step across multiple devices to detect layout inconsistencies, missing elements, and platform-specific rendering bugs.

## Structure

```
pkg/
  regression/   VisualRegression with pairwise comparison, bounded concurrency, JSON response parsing
```

## Key Components

- **`regression.VisionProvider`** -- Interface: Vision(ctx, image, prompt) and SupportsVision(). Must be implemented by consumers
- **`regression.VisualRegression`** -- Main comparator: Compare (single step), CompareMultipleSteps (batch). Uses bounded concurrency via semaphore channel
- **`regression.RegressionResult`** -- Step, StepName, Devices, Differences, Consistent (bool), ComparisonsMade, CriticalCount()
- **`regression.VisualDifference`** -- DeviceA, DeviceB, Description, Severity (Critical/Warning/Info)
- **`regression.DeviceScreenshot`** -- Device, Platform, Screenshot (bytes), Step, StepName
- **Utility functions** -- ConsistencyRate(results), TotalDifferences(results), ValidSeverity(s)

## Data Flow

```
VisualRegression.Compare(ctx, screenshots)
    |
    generate pairwise combinations: (device1, device2), (device1, device3), ...
    |
    for each pair (concurrency-limited by semaphore):
        VisionProvider.Vision(ctx, compositeImage, comparisonPrompt)
            -> parse JSON response -> []VisualDifference
    |
    aggregate -> RegressionResult{Consistent, Differences, ComparisonsMade}

CompareMultipleSteps(ctx, steps) -> Compare() per step -> []*RegressionResult
```

## Dependencies

- `github.com/stretchr/testify` -- Test assertions (only dependency)

## Testing Strategy

Table-driven tests with `testify` and race detection. Tests use mock VisionProvider implementations. Tests cover pairwise comparison generation, concurrency limiting, JSON parsing of vision responses, severity classification, graceful degradation on individual pair errors, and utility function calculations.
