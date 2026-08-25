# Abel-Engineering Cycle Time & Labor Study

A standalone, browser-based engineering tool for capturing cycle-time observations, modeling labor content, reviewing line balance and support workload, and exporting analysis-ready study data.

The application is delivered as a **single HTML file** with inline CSS and JavaScript. It requires no build step, no server, and no network connection for normal use.

**Current runtime version:** `1.30.1`  
**Study schema:** `13`  
**Mapping schema:** `2`  
**Process-template schema:** `5`

> **Version note:** the JavaScript runtime constant reports `1.30.1`, while the visible HTML footer still says `v1.29`. The runtime constant is treated as authoritative in this README.

## What the tool does

The Cycle Time & Labor Study supports end-to-end field studies for paced production lines and shared support work. Its main capabilities include:

- Define the study, reference throughput, conveyor/pitch information, labor-planning basis, and production structure.
- Configure process roles as **Paced / Repeating**, **Recurring / Support**, or **Reference Only**.
- Define process relationships to the selected reference throughput unit, including multi-event and multi-unit relationships.
- Configure recurring support tasks using consumption-based, reference-unit-based, or per-hour frequency models.
- Capture direct timed samples by role, position, operator, task, and sample condition.
- Capture sequenced standard work with ordered work elements for Yamazumi analysis.
- Classify sequence elements as value-added work, work, motion, inspection, wait/idle, machine wait, or other, with independent labor inclusion.
- Record optional line/station observation sessions and contextual events such as starved, blocked, quality hold, machine stop, refill, missed/late, and notes.
- Measure conveyor/belt speed and compare measured speed with configured machine and belt relationships.
- Review mean, standard deviation, P90, operator/position variation, process capacity, labor content, support FTE, and data-quality status.
- Compare measured work with selected production pace, measured bottleneck rate, physical work windows, and recovery time.
- Visualize sequenced work as a **Yamazumi / Standard Work Balance**.
- Run a lightweight staffing/throughput scenario model.
- Export full-fidelity study files, flattened analysis/report JSON, reusable process templates, and Excel timing-sample workbooks.

## Quick start

1. Open the HTML file in a modern browser.
2. Go to **Setup** and enter the study identity, reference throughput/machine pace, and labor-planning assumptions.
3. Define the process roles/stations and any recurring support tasks.
4. Go to **Capture**, select the role, position, operator, and work element/task, then collect timing samples.
5. Use **Review** to validate sample coverage, inspect timing distributions, check process pace/recovery, review events, and inspect labor balance.
6. Use **Data / Export** to download a Study JSON for continuation/audit, a Report JSON for downstream analysis, or an Excel workbook of timing samples.

For repeat studies on the same line, use **Start New Study — Keep Line Setup** to preserve the process configuration while clearing the previous run's observations.

## Main navigation

| Tab | Purpose |
| --- | --- |
| **Overview** | High-level study snapshot, current vs. design labor, area labor summary, quick visuals, Yamazumi access, and next-step indicators. |
| **Setup** | Study identity, throughput/machine pace, labor basis, process roles, support tasks, sequenced standard work, and reusable process templates. |
| **Capture** | Primary field-collection workflow for direct samples, sequenced samples, event context, observation sessions, and recent-sample review. |
| **Review** | Data quality, area/role timing, process capacity, sample review/exclusion, events, operators, process pace, Yamazumi, work-window recovery, and support workload. |
| **Data / Export** | Study lifecycle controls, autosave recovery, Study JSON, Report JSON, timing-sample Excel export, and advanced identity/file-naming information. |

## Process model

### Paced / Repeating roles

Use for work that repeats in relation to the selected reference throughput unit. Each role can define:

- Area
- Current positions/workers
- Process events per reference unit, or reference units per process event
- Optional worker fraction per position
- Optional work-zone length
- Optional reporting/material details
- Optional sequenced standard-work definition

Measured role timing feeds routine labor, position-level load, process capacity, design staffing, pace comparison, and work-window diagnostics.

### Recurring / Support roles

Use for intermittent/shared work such as replenishment, staging, pallet movement, cleanup, paperwork, or other tasks that do not occur as a simple repeated cycle at every production unit.

Support tasks can use one of three recurrence models:

- **Consumption-derived:** consumption per reference unit divided by quantity per service/refill.
- **Reference-unit relationship:** task events per reference unit or reference units per task event.
- **Events per hour:** direct recurrence rate for time-based or irregular support work.

The tool combines measured task duration with modeled recurrence to calculate support labor seconds/hour, labor seconds/reference unit, and design FTE.

### Reference Only roles

Use for process/context definitions that should be retained without contributing as routine paced or support labor.

## Timed capture

The primary timed-sample workflow stores context with each completed sample, including the process role, position, operator identity/alias, timed element or support task, sample condition, and elapsed seconds.

For normal direct timing, tap **Start Timed Sample**, then complete the sample when the observed action ends.

### Sequenced standard-work capture

Paced roles can optionally define an ordered sequence of observable work elements. During sequence capture:

- One tap ends the current element and starts the next from the same boundary timestamp.
- Capture can run as **Continuous Cycles** or **Single Cycle**.
- Sequence revisions are retained with the sample context.
- Wait/idle or machine-wait elements can be recorded as elapsed time without necessarily adding to labor seconds.
- Parent samples retain total elapsed, labor time, wait/idle time, and the per-element breakdown.
- The same sequence data feeds the Yamazumi / Standard Work Balance views and the Work Element Excel export.

The sequence runner can request the browser Screen Wake Lock API when supported so the display is less likely to sleep during active capture.

## Observation and event context

Cycle-time samples work independently from the optional observation-session timer.

The optional observation layer can record broader line/station context, including:

- Starved
- Blocked
- Quality hold
- Machine stop
- Refill
- Missed / late
- Notes

These records support line-loss context, rework/flow review, replenishment review, and downstream reporting without replacing direct cycle-time sampling.

## Engineering review and calculations

The application calculates and displays, where enough inputs/data are available:

- Reference throughput and reference-unit interval
- Mean, standard deviation, and P90 timing
- Paced labor seconds per reference unit
- Support labor seconds per reference unit and per hour
- Current configured crew and crew load
- Theoretical minimum workers
- Design minimum workers using the configured design efficiency
- Role-constrained design positions
- Measured bottleneck throughput
- Remaining/over-pace time by process role
- Physical work-window and recovery reserve
- Shared-support workload and design FTE
- Area/segment labor summaries
- Operator and position timing comparisons
- Replenishment/service-frequency estimates
- Aggregate scenario staffing sensitivity

A central relationship used throughout the tool is:

```text
Reference interval (sec / reference unit) = 60 / reference throughput per minute
```

Routine paced labor is based on measured role time normalized by the process relationship to the reference unit. Support labor combines measured support-task duration with its recurrence model.

## Data-quality and readiness checks

The Review workflow includes heuristic quality checks before a study is treated as a baseline. Examples include:

- Missing reference/machine pace
- No active process roles
- Paced roles with no usable samples
- Support pools/tasks without complete timing or recurrence inputs
- Low sample counts
- Multi-position roles with limited operator coverage
- Active duration events that have not been closed
- Excluded samples/events/speed checks

Current heuristic warning thresholds in the source are:

- **Paced / repeating role:** fewer than **15 normal samples**
- **Included support task:** fewer than **8 normal samples**

These are application readiness heuristics, not a substitute for engineering judgment or a formal sampling standard.

## Data persistence and recovery

The tool stores its working study in browser `localStorage`.

Primary storage keys:

```text
Abel-Engineering_CARTONER_STUDY_V1
Abel-Engineering_CARTONER_STUDY_V1_BACKUP
Abel-Engineering_CARTONER_RUNTIME_RECOVERY_V1
Abel-Engineering_CARTONER_DEVICE_ID
```

Behavior includes:

- Autosave after study changes and capture actions.
- A secondary local backup for smaller study payloads.
- Runtime-recovery metadata for interrupted active sessions/capture states.
- Page-exit persistence for the current study state.
- A visible warning if local persistence fails.
- Capture locking after an autosave failure to reduce the chance of silent data loss.

**Browser storage should not be treated as the only backup.** Download Study JSON regularly during field work, especially before clearing browser data, changing devices, or importing another study.

## Study lifecycle

### Start New Study — Keep Line Setup

Creates a new Study ID and preserves the reusable line configuration, including roles/positions, areas, support-task definitions, machine/pitch setup, design efficiency, plant/line/equipment, and observer defaults.

It clears run-specific data such as operators, timing samples, flow events, observation sessions, belt-speed trials, job/run identity, total headcount, notes, and the report sequence. The next report starts again at `R001`.

### Full Reset

Clears the entire tool back to a blank study, including reusable line/process setup.

## Imports and exports

| Format | Purpose | Includes observations? |
| --- | --- | --- |
| **Study JSON** | Full-fidelity save/resume and audit file. | Yes |
| **Report JSON** | Flattened analysis package for Power BI/downstream labor, timing, pace, quality, and planning workflows. | Yes, flattened for analysis |
| **Timing Samples XLSX** | Human-friendly Excel export of parent timing samples and sequenced work-element samples. | Timing samples only |
| **Process Template JSON** | Reusable Areas, Roles/Stations, and Support Tasks. Imports merge by stable Role/Task ID. | No |
| **Mapping JSON** | Legacy/advanced controlled-dropdown vocabulary retained in the source for compatibility. The normal workflow no longer exposes this as a primary tab. | No |

### Excel timing export

The generated `.xlsx` workbook contains two sheets:

1. **Timing Samples** — parent sample records and study/process context.
2. **Work Element Samples** — element-level rows for sequenced samples.

The workbook is generated directly in the browser; no external spreadsheet library is loaded.

### Report JSON structure

The analysis report currently exports arrays including:

- `Study`
- `Stations`
- `Operators`
- `ObservationSessions`
- `StationObservationSessions`
- `LegacyObservationSessions`
- `BeltSpeedChecks`
- `PlacementSamples`
- `WorkElementSamples`
- `WorkElementSummary`
- `SequenceDefinitions`
- `FlowEvents`
- `SupportTasks`
- `SegmentSummary`
- `ProcessPaceSummary`
- `RoleTimingSummary`
- `OperatorTimingSummary`
- `DataQuality`
- `ScenarioSummary`
- `ExportManifest`

The report schema is `Abel-Engineering.CartonerPowerBIReport`.

### File naming

Study and Report JSON downloads use a database-friendly naming convention:

```text
DATE__PLANT__LINE__JOB__SHIFT__STUDYID__R###__EXPORTUTC__TYPE.json
```

Study-state exports use `R000`. Report exports increment `R001`, `R002`, and so on within the current study.

## Architecture

This is intentionally a compact single-file application:

```text
<single HTML file>
├── inline CSS
├── application markup
└── inline JavaScript
    ├── state and schema migration
    ├── localStorage persistence/recovery
    ├── capture timers and sequence runner
    ├── labor/capacity calculations
    ├── review visualizations
    └── JSON/XLSX export generation
```

There are no external script, stylesheet, package-manager, or build dependencies in the supplied HTML.

### Browser APIs used

The implementation uses standard browser capabilities such as:

- `localStorage`
- `performance.now()` / `requestAnimationFrame()` for timers
- `Blob` and object URLs for downloads
- Clipboard API with a DOM-copy fallback
- `crypto.randomUUID()` when available, with a generated-ID fallback
- Screen Wake Lock for sequenced capture when supported

## Privacy and network behavior

Normal operation is local to the browser. The supplied file has no external scripts/styles and describes itself as requiring no network connection.

Study content remains in browser storage unless the user explicitly downloads, imports, or copies data. There is no built-in backend, account sync, cloud database, or multi-user collaboration layer in this HTML.

## Responsive / field use

The UI includes responsive layouts for tablet/phone widths, large touch-oriented capture controls, horizontal scrolling for dense review tables, mobile sequence-capture layouts, safe-area handling, and a print stylesheet focused on the dashboard/review output.

## Known limitations and interpretation notes

- `localStorage` is device/browser-specific and can be cleared by browser settings or storage policies; use Study JSON for durable transfer/backup.
- An in-progress timed action is not a completed sample. Study export metadata can identify a live snapshot and whether active work/sequence capture was omitted.
- Data-quality thresholds are heuristics and do not certify staffing changes by themselves.
- Work-window/recovery analysis is a physical-feasibility diagnostic and does not replace ergonomic, safety, walking, or overlap observation.
- The quick scenario model is an aggregate sensitivity model; it is not a task-specific simulation of every process change.
- Mapping support remains in the code for compatibility, but the current normal workflow centers on Process Templates rather than the hidden Mapping section.
- There is no server-side persistence, authentication, permissions model, or collaborative editing.
- The visible footer version (`v1.29`) should be updated to match the runtime `APP_VERSION` (`1.30.1`).

## Maintainer notes

When changing the data model or export contracts:

1. Update `APP_VERSION` and, when required, `SCHEMA_VERSION`.
2. Add/adjust migration handling so older Study JSON files remain loadable.
3. Verify Study JSON round-trip import/export.
4. Verify Process Template merge behavior does not replace observations.
5. Verify Report JSON retains expected legacy arrays/field names used downstream.
6. Verify both Excel sheets open correctly and retain the intended parent/element relationship.
7. Test autosave failure/recovery paths and page-exit behavior.
8. Test both desktop and narrow mobile layouts, especially sequenced capture.
9. Update visible version text in the footer when the runtime version changes.

## Source file

This README was prepared from the supplied standalone HTML application:

```text
b484000d-5fc9-4c97-9c6b-d3ef21654ba5.html
```

For a repository, consider renaming the application file to something descriptive such as `cycle-time-labor-study.html` and placing this file beside it as `README.md`.

## License

No license information is declared in the supplied HTML. Add the appropriate project/license terms before distributing the application outside its intended environment.
