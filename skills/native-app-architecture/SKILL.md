---
name: native-app-architecture
description: Organize native app projects, features, files, and module dependencies using feature ownership and consistent Android-style architecture naming. Use when scaffolding an app or feature, deciding where models, repositories, preferences, or use cases belong, reviewing architecture boundaries, or adapting the structure across native platforms. Does not prescribe UI frameworks or shared-code technologies.
---

# Native App Architecture

Organize code around business capabilities. Each feature owns its presentation, business contracts, and data implementation. Core provides reusable mechanisms and a small set of genuinely shared concepts.

Apply these as project conventions, not claims that every platform officially prescribes this architecture. Preserve the user's chosen technologies and existing behavior.

## When and how to use

Use this skill for requests such as:

- “Structure a native app with violation reports and maintenance reports.”
- “Where should this DTO, entity, repository, or preference live?”
- “Split this feature into modules without introducing cycles.”
- “Apply the same architecture to Android, iOS, and desktop.”

Explicit invocation example: “Use $native-app-architecture to organize this feature and explain its dependency boundaries.”

For ordinary implementation work, apply the relevant placement rules without initiating a project-wide restructure. For an architecture review, report evidence and proposed changes; do not move files unless requested.

## Working procedure

1. Inspect local instructions, existing source roots, build targets, imports, visibility, dependency wiring, and tests.
2. Identify the business capability that owns the behavior and its actual consumers. A feature can contain several screens or have no screen.
3. Classify each file using the placement rules below. Separate ownership from storage technology and from physical build layout.
4. Choose the smallest useful package/module boundary. Explain any new module in terms of enforced isolation, reuse, build needs, or team ownership.
5. Show the proposed tree and compile-time dependency direction. For implementation requests, update paths, imports, target membership, resources, composition, and relevant tests together.
6. Check for cycles, implementation leaks, and changed behavior. Run relevant available checks; report what was verified and what remains unverified.

Do not rename an existing project wholesale to match these examples. Use the vocabulary consistently for new organization and migrate existing names only within the requested scope.

## Naming conventions

Use the same architectural vocabulary on every platform:

| Concept | Convention | Example |
|---|---|---|
| App composition | `app` | `app/di`, `app/navigation` |
| Business capability | `feature/<capability>` | `feature/violationreport` |
| Layers | `presentation`, `domain`, `data` | `feature/violationreport/data` |
| Shared capability | `core/<capability>` | `core/network`, `core/datastore` |
| Business model | Meaningful PascalCase noun | `ViolationReport` |
| Transport model | `*Dto` | `ViolationReportDto` |
| Persistence model | `*Entity` | `ViolationReportEntity` |
| Remote endpoint adapter | `*Api` | `ViolationReportApi` |
| Local access object, when needed | `*Dao` | `ViolationReportDao` |
| Repository contract / implementation | `*Repository` / `*RepositoryImpl` | `ViolationReportRepositoryImpl` |
| Preference contract / implementation | `*Preferences` / `*PreferencesImpl` | `ViolationReportPreferencesImpl` |
| Business operation | Verb + concept + `UseCase` | `SubmitViolationReportUseCase` |
| Presentation roles, when needed | `*Screen`, `*ViewModel`, `*UiState` | `ViolationReportUiState` |

Use lowercase package/directory segments such as `violationreport`, `usecase`, `designsystem`, `remote`, `local`, `mapper`, `repository`, `preferences`, `component`, `navigation`, and `di`. Prefer `Dto`, `Api`, `Id`, and `Ui` in application-owned type names for consistency.

Match source filenames to the principal type using the target language's extension and required naming rules. Adapt build identifiers and namespaces to valid platform syntax; for example, logical `core/network` can map to a `CoreNetwork` target. Preserve mandated framework names and generated identifiers. These suffixes describe roles; they do not require one class of every kind.

## Ownership and placement decisions

Ask: “Which capability defines this concept, changes its rules, and owns its lifecycle?” Put the code there first.

| Artifact | Default owner and location | Decision rule |
|---|---|---|
| Screen, state holder, UI state | Feature `presentation/<flow>` | Keep list, details, create, and settings flows within the capability. |
| Domain model | Feature `domain/model` | Express application meaning without transport, database, or UI framework requirements. |
| Repository contract | Feature `domain/repository` | Expose business operations and domain types. |
| Repository implementation | Feature `data/repository` | Coordinate sources, mapping, caching, and data consistency. |
| DTO and endpoint adapter | Feature `data/remote` | Own wire names, serialization, request/response shapes, and transport handling here. |
| Entity and local access | Feature `data/local` | Own persisted representation and feature-specific queries here. |
| Boundary mapper | Feature `data/mapper` or beside its source | Keep dependencies on both representations at the outer data boundary. |
| Preference contract | Feature `domain/preferences` | Describe typed settings and behavior meaningful to that feature. |
| Preference implementation | Feature `data/preferences` | Own keys, encoding, defaults, and feature-specific migrations. |
| Use case | Feature `domain/usecase` | Add for meaningful business rules or orchestration. |
| Client/connection/store setup | Relevant `core` capability | Supply mechanisms without importing feature semantics. |
| Dependency assembly | `app/di`, assisted by feature `di` | Bind contracts to concrete implementations at composition time. |

Create subpackages only when they improve navigation. Small features can put a few contracts directly under `domain` and implementations directly under `data`.

### Sharing a concept

Multiple consumers do not automatically transfer ownership to core. Prefer a narrow public contract from the owning capability, such as a user capability exposing `UserId` or a user summary. Keep profile editing and user persistence within that capability.

Extract to `core/model` only when consumers share the same stable meaning and the model has no dependency on feature internals. Do not merge similarly shaped types that have different business meanings. Never move all feature DTOs and entities to `core/model`.

For a workflow spanning capabilities, give orchestration an explicit owner, such as a coordinating feature or application service. Depend on their contracts; keep generic core independent of those features.

## Core and app responsibilities

Keep `app` focused on entry points, application lifecycle, top-level navigation, configuration, and dependency composition. Keep reusable business rules with their capability.

Create only the core capabilities the app actually needs:

- `core/network`: transport setup, connection monitoring, serialization configuration, and generic transport adapters.
- `core/database`: connection management and generic persistence facilities.
- `core/datastore`: key-value storage construction and access mechanisms. This name does not mandate a particular library.
- `core/designsystem`: theme, typography, and reusable UI primitives.
- `core/ui`: larger shared presentation utilities/components with deliberately limited dependencies. Business-specific widgets stay with their owner or its public UI module.
- `core/model`: justified shared business value types.
- `core/testing`: reusable test infrastructure; production code must not depend on it.

Use named capabilities such as `core/logging` when warranted. Keep `core/common` small if it exists. Avoid catchall `core/data`, `core/domain`, `utils`, `managers`, or `Constants` collections with unrelated ownership.

### Shared database assembly

A single database can store multiple features without owning their business meaning. However, a concrete database declaration that references feature entities has real compile-time dependencies.

If features depend on `core/database`, do not put a feature-aware database aggregator there and introduce a reverse dependency. Assemble the concrete schema in an outer module wired by `app`, or extract feature-owned storage/schema modules that the aggregator can depend on without cycles. Inject the required local access objects into feature data implementations.

Keep feature schema contributions and migrations with their owner where practical; coordinate global schema versions and cross-feature migrations in database assembly. Choose physical storage boundaries for actual transaction and tooling needs, not to mirror the feature tree mechanically.

## Models and repositories

Treat transport, persistence, domain, and presentation as separate responsibilities:

```text
remote DTO <-> data mapping <-> domain model <-> presentation mapping <-> UI state
local entity <-> data mapping <-> domain model
```

Create only representations that exist in the feature. A remote-only feature needs no entity. A UI can consume a domain model directly when it needs no presentation transformation. Do not manufacture duplicate models just to satisfy a diagram.

Keep DTOs and entities behind repository contracts. Map storage and transport errors into application-facing outcomes where needed. Do not expose database handles, HTTP response objects, or storage keys through a domain contract.

The repository implementation selects and coordinates sources; business policies belong in domain logic. A repository can wrap one source and still establish a useful boundary. Avoid an extra data-source interface or mapper class when a small adapter or function is sufficient.

## Preferences ownership

Separate the physical store from the meaning of each setting. Both features may use one store instance while owning separate typed contracts:

| Owner | Example preferences | Private key namespace |
|---|---|---|
| `violationreport` | Include photos by default, confirm before submission, last category ID | `violation_report_*` |
| `maintenancereport` | Default priority, remember last location, last building ID | `maintenance_report_*` |

`ViolationReportPreferencesImpl` and `MaintenanceReportPreferencesImpl` receive storage dependencies from composition. Each owns its keys, encoding, fallback defaults, reset behavior, and migration semantics. `core/datastore` does not know categories, priorities, or reports.

Decide whether preferences belong to a device, user, or account and preserve that scope across account switches. Namespace shared storage keys and coordinate the store's lifetime according to the chosen backend. One file per feature is optional.

A settings screen edits the owning feature's preference contract; it does not take ownership of every setting or access their keys directly. Truly application-wide settings can have a dedicated settings capability. Server-synchronized policy is application data, for example `UserSettingsRepository`, rather than merely local preferences. Credentials belong in an appropriate secure-storage capability.

## Use cases

Add a use case when it encapsulates business validation, policy, a multi-step operation, or reusable orchestration. Examples: `SubmitViolationReportUseCase`, `DetermineMaintenancePriorityUseCase`, and `BuildPlaybackQueueUseCase`.

Allow a presentation state holder to call a repository contract directly for simple reads or writes. Do not add a pass-through use case for every repository method. The `domain` package can contain only models and contracts; a separate domain module and a use-case layer are optional.

## Dependency direction and modularization

Arrows below mean compile-time dependencies, not runtime calls:

```text
app/composition -> feature presentation + feature data + required infrastructure
feature presentation -> feature domain
feature data -> feature domain
feature data -> core/network, core/database, core/datastore as needed
feature presentation -> core/designsystem, core/ui as needed
feature domain -> justified pure shared contracts/models
```

Domain must not import data implementations, presentation, or platform storage/UI frameworks. Presentation uses contracts rather than constructing implementations. Feature implementations must not import sibling feature implementations. Core infrastructure must not import features or app composition. Keep dependencies between core modules acyclic too.

Choose a boundary based on the problem:

| Need | Structure |
|---|---|
| Small app with few boundaries | One build module with packages organized by feature. |
| Feature isolation or independent work | `app`, selected feature modules, and selected core modules. |
| Public feature entry points consumed elsewhere | Optional `feature/<name>/api` and `impl` split. |
| Enforced layer isolation or reuse without UI | Optional feature-owned `presentation`, `domain`, and `data` modules. |

Folders alone do not enforce dependency rules. Use target visibility, module exports, dependency checks, or review according to the project's scale.

For an `api/impl` split, expose only required navigation contracts, operations, or models. Other features depend on `api`; `app` assembles `impl`. Here `api` means a public module boundary, whereas `data/remote/*Api` means a remote service adapter. Keep pure business contracts separate from UI-framework navigation types when independent reuse requires it. Avoid exporting all internals or combining every splitting strategy without a concrete need.

## Example organization

This is a logical source tree. Add the platform's required source roots and file extensions. Omit unused branches; these directories are not automatically separate build modules.

```text
app/
  di/
  navigation/
core/
  network/
  database/
  datastore/
  designsystem/
feature/
  violationreport/
    presentation/
      create/
        CreateViolationReportScreen
        CreateViolationReportViewModel
        CreateViolationReportUiState
      list/
        ViolationReportListScreen
      settings/
        ViolationReportSettingsScreen
    domain/
      model/ViolationReport
      repository/ViolationReportRepository
      preferences/ViolationReportPreferences
      usecase/SubmitViolationReportUseCase
    data/
      remote/ViolationReportApi
      remote/ViolationReportDto
      local/ViolationReportEntity
      local/ViolationReportDao
      mapper/ViolationReportMapper
      repository/ViolationReportRepositoryImpl
      preferences/ViolationReportPreferencesImpl
    di/
  maintenancereport/
    presentation/
    domain/
      model/MaintenanceReport
      repository/MaintenanceReportRepository
      preferences/MaintenanceReportPreferences
    data/
      repository/MaintenanceReportRepositoryImpl
      preferences/MaintenanceReportPreferencesImpl
```

For package-based languages, a logical package could be `com.example.feature.violationreport.data.remote`. Place screen-specific components and state beside the screen; place shared feature components under that feature's `presentation/component`. Keep resources, previews, and tests with their owner using the platform's valid source layout. Keep generated bindings in designated generated sources.

## Cross-platform adaptation

Preserve ownership, layer vocabulary, type roles, and dependency direction across Android, iOS, macOS, Windows, and Linux. Adapt the mechanics to the language and platform:

- Express contracts using native interfaces, protocols, or equivalent abstractions. Use native concurrency, observation, error handling, and lifecycle semantics.
- Map modules to the platform's actual build units. Do not copy Android source-set paths or dependency-injection annotations into unrelated ecosystems.
- Use `*ViewModel` for a presentation state-holder role when one is useful; it does not imply Android inheritance, MVVM for every screen, or identical state management across platforms. A `*Screen` can be implemented with the platform's native view type.
- Keep UI, navigation, lifecycle integration, and platform adapters native. The organizational convention does not select a rendering framework, database, dependency-injection framework, binding generator, or shared language.
- If code sharing is requested, select portable domain/data portions based on actual dependencies. Preserve feature ownership inside the shared code; do not turn `shared` into a catchall layer.
- At a language boundary, use explicit boundary types and adapters. Keep generated foreign-interface types out of unrelated feature UI and domain code where practical; define cancellation, errors, threading, and object ownership according to the selected bridge.

Matching architecture across platforms does not require sharing source code. A physically shared implementation also does not require sharing UI. Explain the chosen boundary before introducing a new sharing technology.

## Do / don't and completion check

| Do | Don't |
|---|---|
| Keep business behavior and its data representations with their owner. | Treat features as UI-only folders and move everything else into core. |
| Expose narrow contracts and inject implementations. | Reach into another feature's repository implementation or storage. |
| Share infrastructure and deliberately shared concepts. | Promote a type to core merely because two files use it. |
| Add modules and use cases for demonstrated boundaries or behavior. | Create one module per directory or one use case per method. |
| Keep preference semantics feature-owned. | Put all keys and feature defaults in a global preferences object. |
| Preserve native implementation semantics with consistent role names. | Force Android lifecycle or framework types onto every platform. |

Before completing, confirm that every new file has a clear owner, public contracts hide implementation types, the module graph is acyclic, and the proposed tree matches real source/target rules. For code changes, check affected builds and tests plus persisted-data compatibility when paths or wiring change. Report the resulting organization, the reasons for non-obvious boundaries, and verification limits concisely.
