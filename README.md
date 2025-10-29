# Compossa Platform

**Building software by composing self-contained, lifecycle-aware components.**

> **📦 Commercial Software** | This repository contains examples and documentation for the Compossa Platform. The platform itself is commercial software distributed as NuGet packages.

---

## What is Compossa?

Compossa is a **composition-based development platform** for building .NET applications and services. At its core, Compossa provides an **`ICompositionHost`** that orchestrates component composition, lifecycle, and dependency ordering.

### Key Characteristics

- **Capability-based architecture** - Components are self-contained, applying resource validation, configuration and on demand enlists into the composition host lifecycle
- **Dependency Moniker system** - Automatic dependency ordering and validation
- **MEF integration** - Components self-describe using System.Composition attributes
- **Cross-platform** - Supports .NET 9+ and bridges from .NET Framework

## Who is Compossa for?

- **Architects** seeking cleaner separation of concerns and maintainable system design
- **Developers** fast up to speed, just cherry pick your building blocks and start adding value by composing your own features
- **Managers** seeking proven approaches that reduce development costs, accelerate time-to-market, and minimize technical debt through composition over monolithic architectures

## Why Compossa?

**Strong Architectural Foundations**

The capability-based architecture with lifecycle management provides:

- **Separation of concerns** - Each capability handles its own setup/teardown
- **Dependency Moniker** - Ensures components are initiated in the correct order
- **Proper resource management** - The LIFO teardown order ensures dependencies are cleaned up correctly
- **Framework integration** - Works with standard .NET hosting patterns

---

## Getting Started

**Access to NuGet Packages:**

The Compossa Platform is distributed as private NuGet packages. To gain access:

1. **Request access** by contacting the repository maintainer
2. You will receive credentials and instructions for configuring your NuGet feed
3. Once configured, install the packages:

## NuGet Packages

### Core Runtime

#### Compossa.Runtime.Abstractions

**Dependencies:** None

Core interfaces and abstractions for the Compossa runtime:

- **`ICompositionHost`** - Composition orchestrator and lifecycle manager
- **`Capability` base class** - Three-phase lifecycle (ConfigureResources → InitializeAsync → TeardownAsync)
- **Dependency Moniker** - Automatic dependency ordering and validation
- **12-state lifecycle** - PowerSaveMode, Calibration, Maintenance, and operational states
- **`IHostedJob`** - Background job scheduling with resource dependencies
- **Runtime/presentation separation** - Deploy to WPF, WinUI, web, or services without code changes

#### Compossa.Runtime.Hosting

**Dependencies:** Compossa.Runtime.Abstractions

Bootstrap and hosting implementation:

- **`Bootstrap.RuntimeAsync()`** - Fluent API for composition host initialization
- **MEF discovery** - Automatic capability scanning via System.Composition attributes
- **Four-phase startup** - Discovery → ConfigureResources → DI Build → Initialize
- **Self-contained packages** - Drop capability assemblies into bin folder for auto-composition
- **Advanced Jobs** - Hosted workflows with resource dependencies and lifecycle integration
- **LIFO teardown** - Automatic reverse-order cleanup

```csharp
var compositionHost = await Bootstrap.RuntimeAsync(configure => configure
    .AddConfigurationSource(config => config.AddEmbeddedJsonFile("appsettings.json"))
    .AddComposition(runtime => runtime.AddRequiredCapability<MyCapability>()),
    new() { Args = args, AutoInitComponents = true });
```

### Presentation Model (MVVM)

#### Compossa.Mvvm.Abstractions

**Dependencies:** Compossa.Runtime.Abstractions

Framework-agnostic MVVM primitives:

- **`IDispatcher`** - Thread affinity abstraction for WPF, WinUI, and Maui
  - Overloaded for up to 7 arguments with return value support
  - `TrySend()`, `TrySendDelayed()` for failure-safe and scheduled operations

- **`IAsyncCommand`** - Async command infrastructure with state management
  - `ICommandState` for XAML binding (`IsBusy`, `IsCompleted`, `IsFaulted`, `IsPaused`, `CanExecute`)
  - `CreatePauseResume()` for commands with pause/resume support
  - `Command<TResult>` for typed results with `OnCompleted()`, `OnError()`, `OnCancelled()` callbacks

- **`PauseToken`** - Suspend/resume primitive for long-running operations
  - `WaitForResumeAsync()` for pause points within async workflows

- **`Validatable` base class** - ViewModels with lifecycle awareness
  - `InitializeAsync(ICommandRegistrar, ...)` for command registration
  - `TeardownAsync()` for cleanup

```csharp
public class ImportViewModel : Validatable
{
    public IAsyncCommand? ImportCommand { get; private set; }

    protected override Task InitializeAsync(ICommandRegistrar commands,
        CancellationToken ct, string[]? args)
    {
        ImportCommand = commands.CreatePauseResume(ImportAsync, CanImport);
        return Task.CompletedTask;
    }

    private async Task ImportAsync(object? param, CancellationToken ct, PauseToken pauseToken)
    {
        foreach (var item in GetItems())
        {
            await pauseToken.WaitForResumeAsync(ct); // Pause point
            await ProcessAsync(item, ct);
        }
    }
}
```

### Diagnostics & Logging

#### Compossa.Diagnostics.Logging

**Dependencies:** Compossa.Runtime.Abstractions, Microsoft.Extensions.Logging

Composition-aware logging integration:

- **MEF-exported `ILogger<T>`** - Automatic logger composition
- **Event ID ranges** - Organized by category (1000-1099 errors, 2000-2099 warnings, 3000-3099 info, 4000-4099 debug)
- **Lifecycle correlation** - Links log entries to capability lifecycle, resource validation, and dependency resolution

#### Compossa.Diagnostics.Serilog

**Dependencies:** Compossa.Diagnostics.Logging, Serilog

Serilog integration reference implementation.

#### Compossa.Diagnostics.Serilog.Console

**Dependencies:** Compossa.Diagnostics.Serilog

Console sink reference implementation.

#### Compossa.Diagnostics.Serilog.File

**Dependencies:** Compossa.Diagnostics.Serilog

File sink reference implementation.

---

## Installation

To gain access to the Compossa Platform:

1. **Request access** by contacting the repository maintainer
2. You will receive credentials and instructions for configuring your NuGet feed
3. Install the packages you need:

```bash
dotnet add package Compossa.Runtime.Abstractions --version 9.0.6
dotnet add package Compossa.Runtime.Hosting --version 9.0.6
dotnet add package Compossa.Mvvm.Abstractions --version 9.0.6
dotnet add package Compossa.Diagnostics.Logging --version 9.0.6
# Add Serilog packages as needed for your logging requirements
```

**Repository Status:**

This repository provides quick reference documentation for the Compossa Platform.

---

**License:** Commercial Software
**Distribution:** NuGet packages only
