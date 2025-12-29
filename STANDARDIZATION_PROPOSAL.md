# Standardization Proposal for ATAS.Indicators.Technical

This document outlines a set of coding standards and best practices for the `ATAS.Indicators.Technical` project. These proposals are based on an analysis of existing indicators (e.g., `SMA.cs`, `RSI.cs`, `ACBW.cs`, `BollingerBands.cs`) and aim to improve code consistency, readability, and maintainability.

## 1. Namespaces and Usings

### Namespace Declaration
Adopt **file-scoped namespaces** (C# 10 feature) to reduce indentation levels and improve readability.

**Proposed:**
```csharp
namespace ATAS.Indicators.Technical;
```

**Avoid:**
```csharp
namespace ATAS.Indicators.Technical
{
    // ... code indented ...
}
```

### Using Directives
- Place `using` directives **outside** the namespace (implicitly required by file-scoped namespaces).
- **Sort** directives:
    1. `System` namespaces first (alphabetical).
    2. Third-party namespaces (alphabetical).
    3. Project namespaces (alphabetical).
- Remove unused `using` directives.

**Example:**
```csharp
using System;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using System.Drawing;

using ATAS.Indicators.Drawing;

using OFT.Attributes;
using OFT.Localization;
```

## 2. Class Structure

### Fields
- Use `private` fields with an underscore prefix and camelCase (e.g., `_myField`).
- Mark fields as `readonly` whenever they are immutable after the constructor (e.g., `DataSeries`, `LineSeries`, constant colors, sub-indicators).
- **Explicit DataSeries Fields**: Define a typed private field for the main `DataSeries` (index 0) instead of accessing it via `DataSeries[0]` or casting it repeatedly. This improves type safety and code clarity.

**Proposed:**
```csharp
private readonly ValueDataSeries _renderSeries = new("RenderSeries", Strings.Visualization);
// ...
// In Constructor:
DataSeries[0] = _renderSeries;
```

### Properties
- Use Auto-Properties where possible.
- **Attributes Order**: Standardize the order of attributes for consistency.
    1. `[Parameter]` (if applicable)
    2. `[Display(ResourceType = typeof(Strings), ...)]`
    3. `[Range]` / `[Mask]` / etc.
- **Localization**: Always use `ResourceType = typeof(Strings)` and `nameof(Strings.ConstantName)` for `Name`, `GroupName`, and `Description`.
- **Grouping**: Use `Strings` constants for `GroupName` (e.g., `Strings.Settings`, `Strings.Visualization`, `Strings.Alerts`).

**Example:**
```csharp
[Parameter]
[Display(ResourceType = typeof(Strings), Name = nameof(Strings.Period), GroupName = nameof(Strings.Settings), Description = nameof(Strings.PeriodDescription), Order = 20)]
[Range(1, 10000)]
public int Period
{
    get => _period;
    set
    {
        _period = Math.Max(1, value);
        RecalculateValues();
    }
}
```

## 3. Constructor
- Initialize `DataSeries[0]` using the private field.
- Add additional series to `DataSeries` or `LineSeries`.
- Set default values for properties.
- **Panel**: Explicitly set `Panel = IndicatorDataProvider.NewPanel;` only if the indicator requires a separate panel (e.g., oscillators).

## 4. OnCalculate Method
- **Initialization Check**: Standardize the check for the first bar at the beginning of the method.
- **Clear Data**: Explicitly clear `DataSeries` on bar 0 if necessary.

**Proposed:**
```csharp
protected override void OnCalculate(int bar, decimal value)
{
    if (bar == 0)
    {
        // Initialization logic
        // DataSeries.ForEach(x => x.Clear()); // If needed
        return;
    }
    // ... calculation logic using _renderSeries[bar] ...
}
```

## 5. Alerts Implementation
Standardize on the "Granular Properties" pattern found in `SMA` and `BollingerBands` for indicators that support alerts. This pattern provides users with explicit control over alert behavior.

**Structure:**
- **Boolean Toggle**: `UseAlerts`
- **Boolean Toggle**: `RepeatAlert`
- **Integer Sensitivity**: `AlertSensitivity` (for approximation alerts)
- **String File**: `AlertFile`
- **Colors**: `FontColor`, `BackgroundColor`

**Grouping**: Use `Strings.Alerts` (or `Strings.ApproximationAlert` for price proximity).

**Example:**
```csharp
[Display(ResourceType = typeof(Strings), Name = nameof(Strings.UseAlerts), GroupName = nameof(Strings.ApproximationAlert), ...)]
public bool UseAlerts { get; set; }

// ... other properties ...

// In OnCalculate:
if (UseAlerts && ... )
{
    AddAlert(AlertFile, InstrumentInfo.Instrument, "Message", BackgroundColor, FontColor);
}
```

## 6. General Coding Style
- **Formatting**: Adhere to standard C# naming conventions and formatting rules (braces, spacing).
- **Comments**: Minimal comments explaining *why*, not *what*. Code should be self-documenting.
- **String Resources**: Avoid hardcoded strings for UI elements; use `OFT.Localization.Strings`.
