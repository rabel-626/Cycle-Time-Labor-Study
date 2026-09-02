# Abel Engineering Cycle Time & Labor Study

Current application version: **1.44.0**  
Current study/report schema version: **16**  
Companion application: `17adfcf4-0039-4d60-91ea-04abcb69c0ab.html`

## Purpose

The Abel Engineering Cycle Time & Labor Study is a standalone browser application for capturing and validating complete-line process timing, labor content, operator variation, production structure, recurring support work, replenishment demand, and line-loss events.

The application is designed to support an industrial-engineering study from initial process definition through timing capture, data-quality review, labor and capacity analysis, and downstream reporting. It runs locally in a modern browser and does not require a server or network connection.

## Current Branding

The application uses the Abel Engineering visual identity:

- Embedded Abel Engineering logo in the application header
- Graphite-black interface with white/silver text
- Electric-violet primary accents derived from the logo
- Responsive desktop and mobile header treatment
- Semantic warning, success, and exception colors retained for operational clarity

The September 2, 2026 branding update was visual only. Calculation logic, element IDs, event wiring, data structures, browser storage behavior, and import/export behavior were not changed.

## Main Workflow

The application is organized into five primary tabs.

### 1. Overview

Provides a high-level study snapshot and access to the main analysis views, including:

- Study identity and readiness status
- Labor-model summary
- Yamazumi and standard-work balance
- Process pace and throughput comparison
- Replenishment summary
- Quick scenario modeling
- Optional flow and line-loss context

### 2. Setup

Defines the study and the process structure before timing begins.

Key setup capabilities include:

- Study name, date, observer, plant, line, job/SKU, shift, equipment, and reference object
- Primary observed output rate
- Two-phase machine-rate measurement using up to 10 saved samples
- Placement-opportunity-window measurement with reference-units-per-window and optional K factor
- Secondary conveyor/belt rate measurement and VFD context
- Labor planning basis and design efficiency
- Areas, process roles, current positions, and worker fractions
- Work models for paced/repeating work, recurring/support work, and reference-only items
- Process-event relationships to the selected reference unit
- Material/component quantity per process event
- Refill quantity and pieces per supply skid
- Optional standard-work sequences for paced roles
- Recurring/auxiliary task definitions and task-level sequences
- Operator aliases and position assignment
- Role display-order controls and a visible Role → Task → Element hierarchy

### 3. Capture

Captures observations against the configured process structure.

Supported capture modes include:

- Standard single-sample timing
- One-tap sequential standard-work timing
- Continuous sequence capture
- One-cycle sequence capture
- Paced-role timing
- Recurring/support-task timing
- Operator and position assignment
- Sample-condition classification
- Line-level and station-level observation sessions
- Quick flow-event logging for conditions such as starved, blocked, quality, rework, machine stop, and waiting
- Duration events and instantaneous events
- Safe cancellation of an active capture or sequence
- Recent-sample review during collection

Sequential capture records the parent elapsed cycle and the individual work-element durations. Wait/idle and non-labor classifications can remain part of elapsed time without automatically becoming labor content.

### 4. Review

Provides detailed analysis and validation before export.

Review functions include:

- Role timing summary
- Operator and position comparison
- Process-capacity review
- Process time versus takt/reference pace
- Labor and elapsed-time Yamazumi views
- Work-element stacks and sequence reconciliation
- Shared support workload
- Recurring-task burden normalized to the reference unit
- Area-level labor summaries
- Replenishment modeling
- Flow/rework event review
- Data-readiness and integrity signals
- Timing outlier review
- Archive, exclusion, reclassification, and reviewer-note controls

### 5. Data / Export

Contains the files and utilities used to save, resume, reuse, and analyze studies.

Available outputs include:

- Full-fidelity Study JSON
- Flattened Analysis Report JSON for Power BI and downstream tools
- Timing Samples Excel export
- Process Template JSON
- Dropdown Mapping JSON
- Copy-to-clipboard options for supported JSON packages

## Process and Labor Model

### Reference-unit normalization

A paced role can be defined in either direction:

- Process events per reference unit
- Reference units per process event

The application normalizes both entry methods to process events per reference unit for labor, capacity, material, and reporting calculations.

### Component consumption

Physical material use is intentionally separated from timing frequency:

`components/reference unit = process events/reference unit × components/process event`

This allows one observed placement cycle to represent multiple components without incorrectly multiplying the measured cycle time.

### Paced labor

Routine paced labor is based on the role timing result and its normalized event frequency. At a high level:

`paced labor/reference unit = role mean seconds × process events/reference unit`

Role positions, worker fractions, reference pace, and design efficiency are then used in load and staffing calculations.

### Recurring and support labor

Recurring tasks can belong to either a paced operator or a shared support labor pool. Supported frequency models include:

- Consumption-driven frequency
- Task events per reference unit or reference units per task event
- Configured events per hour

The resulting burden is normalized into labor seconds per hour, labor seconds per reference unit, and design FTE where applicable.

## Timing Statistics

The study calculates and exports statistics such as:

- Sample count
- Mean
- Median
- Standard deviation
- Coefficient of variation
- P90
- Operator-weighted mean
- Between-operator variation
- Parent labor mean and parent elapsed mean for sequenced work
- Complete, eligible, and incomplete sequence-cycle counts

The selected timing basis can be carried into downstream capacity and material planning.

## Timing Outlier Review

The outlier system is intended to find likely missed clicks and other capture anomalies without silently deleting legitimate observations.

### Comparison groups

Samples are compared only against compatible observations from the same timing group. The baseline uses included, normal-condition, complete observations with positive durations.

Sequential samples are checked at both levels:

- Parent elapsed cycle
- Individual work elements

### Minimum sample count

At least five compatible observations are required before a sample can be flagged.

### Robust comparison method

The application compares each duration with the group median and also calculates a robust score using the median absolute deviation of log-transformed durations.

Current thresholds are:

| Baseline size | Severity | Trigger |
| --- | --- | --- |
| 5–9 samples | Extreme | At least 3× the median or no more than 1/3 of the median |
| 10+ samples | Extreme | At least 3× the median or no more than 1/3 of the median |
| 10+ samples | Strong | At least 2×, no more than 1/2, or robust score ≥ 6 with an additional ratio guard |
| 10+ samples | Watch | At least 1.5×, no more than 2/3, or robust score ≥ 3.5 with an additional ratio guard |

Flags are review prompts, not automatic exclusions. A reviewer can:

- Keep the sample as a valid observation
- Reclassify its condition
- Exclude it as a capture error
- Exclude it for another documented reason
- Reset the review decision

This preserves the raw observation and creates an auditable review trail.

## Yamazumi and Sequence Reconciliation

Standard-work sequences support task-level Yamazumi stacks rather than a single aggregate bar. The application distinguishes:

- Labor balance
- Elapsed cycle composition
- Wait/idle content
- Non-labor content
- Recurring auxiliary work
- Complete versus incomplete sequence cycles
- Parent-cycle versus element-stack reconciliation

The parent sample remains authoritative. Element totals are reconciled against complete cycles from the current sequence revision so incomplete or obsolete sequence definitions are not silently mixed into the analysis.

## Data Files and Schemas

### Study JSON

- Schema: `Abel-Engineering.CartonerStudy`
- Schema version: `16`
- Purpose: full-fidelity save/resume and audit file
- Contains setup, roles, tasks, operators, samples, events, sessions, rate checks, review decisions, and migration history

### Analysis Report JSON

- Schema: `Abel-Engineering.CartonerPowerBIReport`
- Schema version: `16`
- Purpose: flattened analysis package for Power BI, multi-study analysis, and the Material Flow & Takt Planner

Major report arrays include Study, Stations, Operators, ObservationSessions, StationObservationSessions, BeltSpeedChecks, OpportunityWindowChecks, MachineRateChecks, PlacementSamples, WorkElementSamples, SequenceDefinitions, FlowEvents, SupportTasks, SegmentSummary, ProcessPaceSummary, RoleTimingSummary, OperatorTimingSummary, SequenceReconciliation, DataQuality, DataAccuracy, ScenarioSummary, and ExportManifest.

### Process Template JSON

- Schema: `Abel-Engineering.CycleTimeLaborRoleStations`
- Schema version: `5`
- Purpose: reuse areas, roles/stations, support tasks, and their configured standard-work structures without carrying timing observations

Process Template import merges definitions by stable Role/Task ID. Matching definitions are updated, new definitions are added, and definitions absent from the imported file are retained. Samples, events, operators, and observation history are not replaced.

### Dropdown Mapping JSON

- Schema: `Abel-Engineering.CartonerMapping`
- Schema version: `2`
- Purpose: reuse dropdown standards without replacing the active study or process data

## File Naming Convention

Study and report downloads use the database-friendly convention:

`DATE__PLANT__LINE__JOB__SHIFT__STUDYID__R###__EXPORTUTC__TYPE.json`

- Study-state files use `R000`.
- Report exports increment `R001`, `R002`, and so on within the study.
- Starting a new study resets the next report number to `R001`.

## Local Storage and Recovery

The application stores working data in the browser using local storage.

Primary storage keys include:

- `Abel-Engineering_CARTONER_STUDY_V1`
- `Abel-Engineering_CARTONER_STUDY_V1_BACKUP`
- `Abel-Engineering_CARTONER_RUNTIME_RECOVERY_V1`
- `Abel-Engineering_CARTONER_DEVICE_ID`

The application maintains a primary autosave, a rolling backup when practical, and runtime-recovery information for interrupted observation timers. If autosave fails, capture is locked to prevent silent data loss and the operator is directed to export the Study JSON.

Browser local storage is not a substitute for a controlled study file. Clearing browser data, changing browsers, changing browser profiles, or opening the file under a different origin can make an existing autosave unavailable.

## Recommended Operating Procedure

1. Open the HTML file in a current desktop or mobile browser with JavaScript enabled.
2. Complete Study Identity and Reference Pace setup.
3. Define areas, roles, support pools, recurring tasks, and optional sequences.
4. Verify the Role → Task → Element hierarchy before collection.
5. Record operator aliases and position assignments.
6. Capture normal cycles first, then deliberately classify abnormal conditions and events.
7. Review incomplete sequences, outliers, exclusions, and readiness warnings.
8. Download the full Study JSON as the authoritative continuation file.
9. Download the Analysis Report JSON for Power BI or Material Flow & Takt planning.
10. Retain exported files in a controlled study folder with the generated filenames unchanged.

## Backup and Data-Safety Guidance

- Export Study JSON at the beginning and end of each major collection session.
- Export again before clearing browser storage, switching devices, or replacing the HTML application.
- Do not treat Report JSON as a substitute for Study JSON; the report is flattened for analysis and is not the authoritative continuation file.
- Review every automated outlier flag before exclusion.
- Preserve exclusion reasons and reviewer notes for auditability.
- Confirm reference-unit relationships and component quantities before relying on staffing or material calculations.

## Browser Requirements

- Current Chromium-based browser, Firefox, or Safari
- JavaScript enabled
- Local storage enabled
- File-download permission for JSON/Excel exports
- Pop-up/print permission when using browser print functions

No installation, web server, database connection, or internet connection is required for normal operation.

## Functional Boundaries

- Results are only as reliable as the process structure, reference-rate assumptions, sample classification, and observation quality entered by the user.
- Outlier detection identifies statistically unusual observations; it does not determine whether an observation is operationally invalid.
- Staffing and capacity outputs are engineering planning estimates and should be checked against safety, ergonomics, equipment limits, material availability, and actual production trials.
- Local browser storage is device/profile specific and should not be treated as the only record of a completed study.

## Current Change Summary

### September 2, 2026 — Abel Engineering visual refresh

- Added the supplied Abel Engineering logo to the application header.
- Embedded a compact logo asset directly in the HTML for standalone use.
- Replaced the prior multicolor primary interface accents with a black, silver/white, and violet brand system.
- Updated active tabs, panels, inputs, information controls, gradients, and footer branding.
- Preserved semantic status colors for warnings, valid states, and exceptions.
- Made no changes to functional JavaScript, calculations, IDs, event handlers, schemas, or stored data.

### Functional baseline represented by this README

- Application version 1.44.0
- Study/report schema 16
- Mapping schema 2
- Process Template schema 5
- Role → Task → Element hierarchy
- Recurring-task and component-consumption modeling
- Sequential standard-work capture and Yamazumi reconciliation
- Two-phase machine-rate measurement
- Placement-opportunity and belt-rate verification
- Timing outlier review with auditable decisions
- Power BI, Excel, Process Template, Mapping, and Study exports

