#Abel-Engineering Cartoner Labor Study

A standalone, phone-first industrial engineering tool for measuring labor content, operator variation, line losses, replenishment demand, rework burden, and staffing requirements on machine-paced cartoner assembly lines.

The application runs entirely in the browser as a single HTML file. It is designed for floor studies where multiple operators may perform the same repeated role—such as booster-pack placement, promo-card placement, shell loading, carton base/lid placement, QC, or rework—and where the long-term goal is to build a reliable baseline for labor optimization, automation studies, and Power BI reporting.

## Current Version

- **App:** 1.4.0
- **Study schema:** 3
- **Format:** Standalone HTML / JavaScript
- **Storage:** Browser local storage with JSON import/export
- **Network requirement:** None for normal use

## Why This Tool Exists

Traditional stopwatch studies can capture individual cycle times, but they often lose the context needed to answer broader engineering questions:

- How much labor does each cartoner role actually consume?
- How much variation exists between operators performing the same work?
- Which roles are responsible for most of the direct labor?
- How many workers are theoretically required at the current cartoner rate?
- How many workers are required after applying a realistic design-efficiency allowance?
- Does the current role layout impose staffing constraints that a simple total-labor calculation misses?
- How often is the line starved, blocked, held for quality, or affected by rework?
- How much material is consumed at each station and how often should replenishment occur?
- Which manual operations are the best candidates for future automation?
- Can data from many studies, lines, products, observers, and devices be combined cleanly in Power BI?

Abel-Engineering Cartoner Labor Study is intended to preserve the raw observations needed to answer those questions instead of only storing final averages.

## Core Study Structure

The application organizes captured work using the following hierarchy:

```text
Study
└── Role / Station Group
    ├── Physical Position
    ├── Operator Alias
    ├── Work Element
    ├── Timed Samples
    └── Events / Exceptions
```

Examples of roles include:

- Thermoformed Shell Load
- Cartoner Base Placement
- Cartoner Lid Placement
- Booster Pack Placement
- Promo Card Placement
- Tin Insert Placement
- Large Card / Oversized Insert Placement
- QC / Cosmetic Inspection
- Rework
- Material Replenishment
- Machine Interaction

A role can have multiple simultaneous positions. For example, one product may require six workers performing the same Booster Pack B placement operation. Samples can therefore be compared by role, physical position, and operator without creating six independent process definitions.

## Phone-First Capture

The capture interface is designed for use on a phone while observing work on the production floor.

### Observation timer

Use **START STUDY** to begin the overall observation window and **PAUSE STUDY** when the observer stops watching the process.

This creates an auditable total observation duration and allows line-loss events to be normalized against actual observed time.

### Timed work-element capture

For a selected role, position, operator, and work element:

1. Tap **START WORK ELEMENT** when the action begins.
2. Tap **COMPLETE WORK ELEMENT** when the action ends.
3. Repeat for the desired number of samples.

The visible timer is intentionally limited to tenths of a second to reduce distraction, while the saved duration retains higher precision.

### Sample conditions

Timed samples can be classified as:

- Normal
- Material difficulty
- Misfeed
- Operator interruption
- Other exception

Normal samples are preferred when calculating role timing standards. Exception samples remain preserved in the raw data for later analysis.

## Operator Comparison

Observers enter a simple non-sensitive alias such as:

```text
OP-01
OP-02
OP-03
```

The alias only needs to be consistent within the study.

Internally, the application creates a unique **OperatorObservationID** so that `OP-01` in one study is not automatically treated as the same person as `OP-01` in another study or on another device.

The tool calculates role- and operator-level statistics including:

- Sample count
- Mean
- Median
- Sample standard deviation
- Coefficient of variation (CV)
- P90
- Operator-weighted mean
- Between-operator standard deviation

This allows engineering analysis to distinguish task variability from operator-to-operator variability.

## Role-Based Labor Model

Different work types are not blended into one generic placement average.

For each routine role:

```text
Role Labor sec/carton
    = Role Mean Time × Required Quantity per Carton
```

Total modeled labor is:

```text
Total Labor sec/carton
    = Σ Routine Role Labor
    + Rework Labor
    + Unmodeled Fixed Labor
```

This supports products where shell loading, booster placement, promo cards, lid placement, QC, and other operations have materially different cycle times.

## Machine Pace

Machine pace can be entered directly as **Cartons per Minute**, or calculated from conveyor speed and carton pitch.

```text
Carton Interval (sec/carton)
    = 60 ÷ Cartons per Minute
```

When direct cartons/min is not entered:

```text
Cartons per Minute
    = Conveyor Speed ÷ Carton Pitch
```

Units are converted internally before calculation.

## Staffing Calculations

### Theoretical minimum

The mathematical lower bound assumes perfectly balanced labor:

```text
Theoretical Minimum Workers
    = CEILING(Total Labor sec/carton ÷ Carton Interval)
```

### Design minimum

A configurable design-efficiency allowance provides reserve for normal operating variation:

```text
Design Minimum Workers
    = CEILING(
        Total Labor sec/carton
        ÷ (Carton Interval × Design Efficiency)
      )
```

### Role-constrained design

Each routine role is also sized independently:

```text
Role Design Positions
    = CEILING(
        Role Labor sec/carton
        ÷ (Carton Interval × Design Efficiency)
      )
```

The application then sums the role-level requirements.

This is important because total labor may suggest that fewer people are theoretically possible even when individual station assignments, reach zones, task precedence, or component-specific requirements prevent perfect rebalance.

## Observation Losses and Events

The tool separates instantaneous events from duration events.

### Duration events

- Starved
- Blocked
- Quality Hold
- Rework

Tap once to start the event and again to end it.

The study observation timer continues running, allowing the application to calculate total observed time and normal-running time.

```text
Normal Running Time
    = Total Observed Time
    - Starved Time
    - Blocked Time
    - Quality Hold Time
```

### Instantaneous events

- Refill
- Missed / Late Placement
- Note

Each event is tagged to the current role, physical position, and operator assignment when available.

## Rework

Rework is treated as exception-driven labor rather than automatically adding a fixed rework time to every carton.

```text
Estimated Cartons Observed
    = Cartons per Minute × Normal Running Minutes
```

```text
Rework Labor sec/carton
    = Total Observed Rework Duration
    ÷ Estimated Cartons Observed
```

The resulting data can be used to evaluate both rework recurrence and the labor burden created by rework.

## Material Consumption and Replenishment

Each role can optionally define:

- Material / component
- Quantity used per carton
- Quantity per refill
- Work-zone length
- Worker fraction

Material consumption is calculated as:

```text
Consumption pieces/min
    = Cartons per Minute × Quantity per Carton
```

Expected replenishment frequency is:

```text
Expected Refills/hour
    = (Cartons per Minute × Quantity per Carton ÷ Quantity per Refill) × 60
```

Observed refill events can then be compared with the expected replenishment demand.

## Work-Window Estimate

When conveyor speed and usable work-zone length are known:

```text
Work Window (sec)
    = Work-Zone Length ÷ Conveyor Speed × 60
```

This supports future analysis of whether a mathematically balanced role assignment is physically achievable within the available conveyor reach.

## Cross-Study Standardization

Each role contains both a floor-facing role name and standardized reporting dimensions.

### Standard Role

Examples:

- `THERMOFORMED_SHELL_LOAD`
- `CARTONER_BASE_PLACEMENT`
- `CARTONER_LID_PLACEMENT`
- `BOOSTER_PACK_PLACEMENT`
- `PROMO_CARD_PLACEMENT`
- `TIN_INSERT_PLACEMENT`
- `LARGE_CARD_PLACEMENT`
- `QC_INSPECTION`
- `REWORK`
- `MATERIAL_REPLENISHMENT`
- `MACHINE_INTERACTION`

### Component Family

Examples:

- `THERMOFORMED_SHELL`
- `CARTONER_BASE`
- `CARTONER_LID`
- `BOOSTER_PACK`
- `PROMO_CARD`
- `TIN_INSERT`
- `LARGE_CARD`
- `QC`
- `REWORK`
- `MATERIAL`
- `MACHINE`

This allows a specific role such as **Booster Pack B Placement** to retain its product-specific name while still being grouped with other booster-pack studies in Power BI.

## Stable Record Identity

The application generates unique identifiers for the major data entities, including:

- Study
- Device instance
- Role
- Operator observation
- Timed sample
- Event
- Observation session

The IDs are intended to prevent record collisions when JSON exports from different users or phones are later appended into a central dataset.

## Study Lifecycle

Each study has a status:

- Draft
- In Progress
- Complete
- Reviewed
- Approved

The status does not alter timing calculations. It exists so downstream reports can distinguish experimental or incomplete studies from reviewed baseline data.

## Data Preservation

The application is designed to preserve raw observations.

### Samples and events

Incorrect samples or events are **excluded**, not permanently removed from the study dataset.

Excluded records:

- remain in Study JSON
- remain available for audit
- can be restored
- are ignored by current timing and staffing calculations

### Roles

Roles with historical data can be archived instead of being removed from the active model.

This approach keeps the engineering audit trail intact as the study evolves.

## Study Readiness and Data Quality

The dashboard includes a lightweight readiness check.

Current heuristics include:

- Valid machine pace is required.
- At least one active role should be configured.
- Routine roles should contain usable timed samples.
- A routine role with fewer than **15 normal samples** is flagged.
- A multi-position routine role should include at least **2 operator aliases**.
- An observation window shorter than **10 minutes** is flagged.
- Active duration events must be closed before the study is considered ready.
- Unmapped standard roles/component families are flagged for cross-study reporting quality.

These are screening heuristics, **not formal approval criteria or statistical guarantees**. Engineering review is still required before staffing or automation decisions are implemented.

## Autosave and Offline Use

The app operates entirely in the browser and requires no network connection.

Current persistence uses browser `localStorage`.

The application stores:

- editable study state
- study identity
- role definitions
- operators
- samples
- events
- observation sessions
- configuration

Because the current build uses local browser storage, it should not be treated as the final multi-user system of record. Export study data regularly.

## JSON Export

Two JSON formats are available.

### Editable Study JSON

Full-fidelity application state intended for:

- continuing a study
- transferring a study between devices
- auditing source observations
- schema migration
- preserving raw data

### Power BI Report JSON

A reporting-oriented package containing flattened tables:

- `Study`
- `Stations`
- `Operators`
- `ObservationSessions`
- `PlacementSamples`
- `FlowEvents`
- `RoleTimingSummary`
- `OperatorTimingSummary`
- `DataQuality`
- `ScenarioSummary`
- `ExportManifest`

Stable Study, Role, Operator, Sample, Event, and Session identifiers are intended to support safe append across many exported studies.

Both Study JSON and Power BI JSON can be downloaded or copied directly to the clipboard for transfer from a phone.

## Scenario Planner

The scenario planner estimates labor requirements for alternate placement-item counts and cartoner speeds.

The placement portion of measured labor scales with the scenario item count while non-placement work such as loading, lid/base work, QC, and observed rework remains in the baseline.

This is intended to help estimate staffing for products with longer or shorter bills of material without requiring an entirely new first-principles study for every configuration.

## Field-Level Help

Data-entry fields include information buttons that explain:

- what should be entered
- how the field is used
- whether it affects calculations or is reference-only
- the relevant calculation when applicable

The help interface is responsive and changes to a phone-friendly bottom-sheet layout on small screens.

## Basic Workflow

1. Open the HTML file in a modern browser.
2. Complete **Study & Line Setup**.
3. Enter the validated machine pace.
4. Define each cartoner **Role / Station Group**.
5. Map roles to Standard Roles and Component Families when possible.
6. Set current positions and required quantity/carton.
7. Open **Capture**.
8. Select the role, physical position, and operator alias.
9. Start the study observation timer.
10. Capture repeated work-element samples.
11. Log starvation, blocking, quality holds, rework, refills, and other exceptions when observed.
12. Repeat observations across positions/operators.
13. Review the dashboard, role statistics, data-quality warnings, and staffing calculations.
14. Mark the study Complete/Reviewed/Approved according to the applicable internal process.
15. Export the editable Study JSON.
16. Export the Power BI Report JSON for aggregation.

## Running the Application

No build process or installation is currently required.

```text
Abel-Engineering_Cartoner_Labor_Study_v1.4.html
```

Open the file directly in a current browser.

The application is intentionally self-contained and does not require an external server for normal operation.

## Power BI Direction

The reporting JSON is designed so many studies can eventually be collected into a central repository and appended into Power BI.

A future production architecture may look like:

```text
Phone / Browser Study App
        ↓
Central Study Repository
        ↓
Power BI Dataset / Semantic Model
        ↓
Engineering & Operations Dashboards
```

The present application provides the baseline capture and export structure; it does **not** yet provide automatic cloud synchronization, authentication, or a shared multi-user database.

## Long-Term Goal

The long-term goal is to create a reliable manual-process baseline that supports:

- labor optimization
- line balancing
- operator-variation analysis
- material-flow analysis
- rework reduction
- automation opportunity identification
- automation challenge identification
- before/after automation validation
- multi-line and multi-product benchmarking
- Power BI trend analysis
- eventual transition from manual study capture to increasingly autonomous data collection

The tool is intended to answer not only **how many workers are present**, but **where labor is consumed, how stable each operation is, what losses are occurring, and which manual processes offer the strongest automation opportunity**.

## Current Limitations

The current version is still an engineering study tool rather than a production MES or centrally hosted application.

Not currently included:

- centralized cloud database
- user authentication
- automatic multi-device synchronization
- automatic Power BI publishing
- centralized role master-data administration
- persistent enterprise operator registry
- direct machine/PLC data collection
- automatic vision-based work-element recognition
- formal statistical approval workflow

These are intentionally deferred until real floor studies validate the baseline workflow and data model.

## Engineering Note

The staffing and readiness calculations are decision-support tools. They should be combined with floor validation, ergonomic constraints, task precedence, work-zone limitations, material-flow requirements, safety requirements, quality requirements, and normal process variation before implementing staffing reductions or automation changes.
