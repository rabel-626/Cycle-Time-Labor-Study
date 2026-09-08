---
documentId: WI-AE-CTL-001
title: Cycle Time & Labor Study — Study Procedure
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Cycle Time & Labor Study — Study Procedure

## 1. Purpose

This work instruction defines the recommended operating procedure for creating, executing, reviewing, and saving a study in the **Abel Engineering Cycle Time & Labor Study** application.

The procedure is intended to preserve traceability between:

- the observed production condition,
- the process structure entered into the application,
- the raw timing observations,
- the normalized labor model,
- the review decisions applied to samples, and
- the saved study/report package used downstream.

> **Scope boundary:** The Cycle Time & Labor Study is intended for process timing, labor-content, work-balance, recurring-work, and capacity analysis. Formal line downtime and operating-loss analysis belongs in the Abel Engineering **Downtime Tracker**.

---

## 2. Scope

Use this procedure when performing a new or revised cycle-time/labor study for a production line, station group, or process where the application will be used to model one or more of the following:

- repeated paced work,
- multiple physical positions performing the same role,
- standard-work element sequences,
- operator-to-operator or position-to-position variation,
- recurring side work assigned to paced roles,
- shared support/material-handling workloads,
- component consumption and replenishment frequency,
- process capacity relative to a production reference rate,
- labor-content and staffing estimates,
- Yamazumi / line-balance analysis.

---

## 3. Responsibilities

### 3.1 Observer / Study Owner

The observer is responsible for:

- defining what one timing observation represents,
- entering the correct study identity and production reference,
- configuring roles, positions, process relationships, and recurring tasks,
- consistently identifying sample condition and operator/position context,
- avoiding intentional sample selection that biases the result,
- preserving abnormal-but-valid observations instead of deleting them solely because they are unusual,
- saving the study to the controlled Study Repository.

### 3.2 Reviewer / Engineer

The reviewer is responsible for:

- checking the configured process model against the physical process,
- reviewing sample coverage, variation, outliers, and sequence reconciliation,
- verifying recurrence assumptions and process-event relationships,
- determining whether exclusions are justified,
- interpreting staffing and capacity results in the context of safety, ergonomics, quality, equipment, skills, and operating constraints.

The observer and reviewer may be the same person when local practice permits, but the two responsibilities remain distinct.

---

## 4. Definitions

| Term | Definition |
| --- | --- |
| **Reference unit** | Finished or common production unit used as the normalization basis for throughput, labor, and material calculations. |
| **Primary Observed Output Rate** | Validated finished-reference-unit throughput used as the primary staffing/reference basis. |
| **Process event** | One occurrence of the timed action represented by a process role. |
| **Paced role** | Repeated production work whose demand scales with the reference-unit flow. |
| **Recurring / side task** | Lower-frequency work with its own measured duration and recurrence model. |
| **Support pool** | One or more positions that share intermittent recurring tasks rather than performing one action every reference cycle. |
| **Physical position** | One parallel instance of a role, such as Position 1 and Position 2 on two sides of a line. |
| **Normal sample** | Observation classified as representative of normal production for the defined process. |
| **Parent cycle** | Complete observed cycle stored as the authoritative timing sample for a sequenced role/task. |
| **Sequence element** | Ordered child detail within a parent cycle. |
| **Design efficiency** | Planned utilization ceiling used to convert labor content into design staffing. |
| **Study Repository** | Approved local repository folder used by the current application for official save/open operations. |

---

## 5. Prerequisites

Before floor collection begins:

1. Use the current approved Cycle Time & Labor Study HTML.
2. Use desktop Microsoft Edge or Google Chrome when the official Study Repository save/open workflow is required.
3. Confirm access to the approved repository root.
4. Identify the study scope and reference unit.
5. Establish or measure a valid primary production throughput.
6. Understand the physical process well enough to define repeated roles and meaningful sample boundaries.
7. Determine whether recurring work or standard-work sequences need to be modeled.
8. Establish non-sensitive operator aliases if operator variation is being evaluated.

> **Warning:** Do not begin collecting large quantities of timing data before confirming the process-event/reference-unit relationships. An incorrectly modeled relationship can make otherwise accurate stopwatch observations produce incorrect labor and capacity outputs.

---

# 6. Procedure

## 6.1 Open or Create the Study

### New study

Open the application and confirm that no prior study needs to be preserved.

If a previous study is present, use **SAVE STUDY** before starting a new record.

### Resume study

Open **Data / Export → OPEN STUDY** and select the correct repository record.

When opening a different Study ID, review the warning before replacing the current browser autosave.

### Legacy study

Use **LEGACY FILE LOAD** only for older Study JSON files outside the Study Repository. After loading a legacy study, use **SAVE STUDY** so the current record is written into `01_CYCLE_TIME`.

---

## 6.2 Connect the Study Repository

1. Open **Data / Export**.
2. Select **CONNECT / RECONNECT REPOSITORY**.
3. Choose a repository root named exactly:
   - `Engineering Study Hub - Study Repository`, or
   - `Study Repository`.
4. Grant read/write permission when prompted.
5. Confirm that the repository status indicates a successful connection.

The Cycle Time route is:

```text
Study Repository / 01_CYCLE_TIME / PLANT / LINE / YYYY / YYYY-MM / CT-RECORD-ID /
```

The application performs a writeability check when connecting/saving. Direct file saves are verified locally. Any OneDrive/SharePoint/Teams synchronization is handled outside the application by the configured OneDrive client.

---

## 6.3 Complete Study Identity

In **Setup → Edit Study**, complete the applicable identity fields:

- Study name
- Date
- Study status
- Observer
- Plant
- Line
- Reference object
- Job/SKU
- Shift
- Equipment
- Total headcount
- Notes

Use **Draft** while constructing the study and **In Progress** while collecting data. Use later status values only in accordance with the site's review/approval practice.

### Data-quality rule

Study identity fields do not usually change the timing mathematics, but they are critical for:

- traceability,
- repository path/naming,
- report filtering,
- comparing studies across products or dates,
- determining whether two studies represent comparable conditions.

---

## 6.4 Define the Reference Unit and Primary Throughput

Select a reference unit that is meaningful across the process. Enter the **Primary Observed Output Rate** in reference units per minute.

```text
Reference interval = 60 / Primary reference rate
```

Example:

```text
Rate = 15 reference units/min
Interval = 60 / 15 = 4.00 sec/reference unit
```

The application treats a missing primary rate as a **readiness blocker** for staffing results.

### Optional rate checks

Use the available validation methods where useful:

- direct/two-phase machine-rate measurement,
- conveyor/belt speed and pitch,
- placement-opportunity-window measurement.

These provide comparison/diagnostic rates. Do not substitute one for the primary staffing rate unless the study owner intentionally changes the Primary Observed Output Rate after engineering review.

---

## 6.5 Configure Design Efficiency

Enter the intended **Design Efficiency %**.

Example: `85%` means planned modeled labor should consume no more than 85% of the theoretical available labor time in the aggregate design calculation.

```text
Aggregate design workers
= CEILING(total labor sec/reference unit
  / [reference interval × design efficiency])
```

Design efficiency does **not** change measured cycle times. It changes the staffing allowance applied to them.

---

## 6.6 Define Areas

Create meaningful Areas to organize the line for setup, capture filtering, visual review, and reporting.

Area is primarily an organizational/reporting dimension. A missing Area does not invalidate the timing arithmetic, but the application flags missing Area assignments as a warning because grouping and review become less clear.

---

## 6.7 Define Process Roles

For each repeated production or support function, create a Process Role.

Confirm:

- role name/category,
- Area,
- work model,
- current positions,
- worker fraction,
- headcount inclusion,
- routine labor inclusion,
- core timed element description,
- reference-unit relationship,
- component relationship if material modeling is required.

### Choose the correct work model

**Paced / repeating role** — Use when work demand scales with finished-unit production.

**Recurring / support role** — Use for a shared pool that services intermittent tasks.

**Reference-only role** — Use where a process needs to be visible/documented but should not contribute as routine modeled labor.

> **Engineering note:** Do not combine fundamentally different operations into one role simply to reduce setup effort. A timing mean is only meaningful when the observations represent the same defined work.

---

## 6.8 Define the Process-to-Reference Relationship

For each paced role, define frequency in either direction:

- process events per reference unit, or
- reference units per process event.

The application normalizes both to:

```text
E_ref = process events/reference unit
```

If `u` reference units are produced for one process event:

```text
E_ref = 1 / u
```

### Example A — multiple placements per unit

If a worker performs 6 placements for every finished carton:

```text
E_ref = 6 events/carton
```

### Example B — one process event covers multiple units

If one process event serves 3 cartons:

```text
E_ref = 1/3 = 0.3333 events/carton
```

This relationship is used in labor, capacity, material, and report normalization.

---

## 6.9 Configure Component Consumption Where Needed

Material quantity is intentionally separate from timing frequency.

```text
Components/reference unit
= process events/reference unit × components/process event
```

Example:

```text
2 process events/carton × 3 components/event
= 6 components/carton
```

Do not multiply the observed stopwatch duration by component quantity unless the timed event itself actually repeats for each component. The process-event relationship and material quantity serve different purposes.

---

## 6.10 Configure Recurring / Side Tasks

Create recurring tasks for intermittent labor that should not be forced into every core cycle.

For each task define:

- owning paced role or support pool,
- task name/category,
- whether it contributes to labor,
- timing method or optional task sequence,
- recurrence method,
- recurrence quantity/frequency,
- interruption behavior when relevant.

### Recurrence methods

#### Consumption driven

```text
Events/reference unit
= component demand/reference unit / quantity per service event
```

#### Reference-unit recurrence

Enter either events/reference unit or reference units/task event. The application normalizes the relationship.

#### Events per hour

```text
Events/reference unit
= events/hour / (reference units/min × 60)
```

### Recurring-task labor

```text
Labor sec/hour = mean task sec × events/hour
```

```text
Labor sec/reference unit = mean task sec × events/reference unit
```

> **Important:** A recurring task that interrupts a paced operator may create a short-term feasibility problem even when its average labor seconds fit mathematically. Verify buffer, coverage, natural recovery, or task timing separately.

---

## 6.11 Define a Standard-Work Sequence When Appropriate

Use a sequence for a paced role or recurring task when the cycle should be decomposed into an ordered set of work elements.

For each element:

- define a concise action name,
- maintain a stable order,
- identify whether the element is labor-included,
- identify wait/non-labor content accurately.

### Sequence data rule

One completed observation remains one **parent cycle**. Child elements describe that cycle.

Do not interpret each element as an independent process cycle.

### Revision rule

When the sequence definition changes, the application uses the **current sequence revision** for reconciliation. Old sequence definitions are retained in the data record but are not silently mixed into the current standard.

---

## 6.12 Assign Operator Aliases and Positions

Before each observation:

1. Select the actual **Physical Position**.
2. Enter/reuse the same alias for the same observed person during the study.
3. Avoid names or personally identifying information when an anonymous alias is sufficient.

For multi-position roles, distribute observations across the process rather than timing only the easiest or most accessible position.

The application's readiness review warns when a role has multiple positions but fewer than two operator aliases represented in the selected timing basis.

---

## 6.13 Capture Normal Core Work

For one-shot timing:

1. Select the Process / Role.
2. Leave Recurring / Side Task blank.
3. Confirm Physical Position and Operator Alias.
4. Confirm **Normal** condition if the process is representative.
5. Start **CAPTURE WINDOW**.
6. Stop at the consistently defined cycle boundary.
7. Review the recent sample before continuing.

Define the cycle boundary consistently across observations. Examples of good boundaries are specific, repeated physical events—not subjective judgments such as “when the task feels finished.”

---

## 6.14 Capture Sequential Standard Work

For an enabled sequence:

1. Select the sequenced role/task.
2. Select **Continuous Cycles** or **Single Cycle**.
3. Start the sequence.
4. At each work-element boundary, use **END CURRENT / START NEXT**.
5. Use **UNDO LAST** if the most recent boundary was accidental.
6. Use **STOP AFTER THIS CYCLE** when ending a continuous run cleanly.
7. Use **CANCEL / DISCARD** if the current cycle is invalid/incomplete and should not be saved as a completed observation.

The application captures:

- parent labor seconds,
- parent elapsed seconds,
- individual element durations,
- sequence identity/revision,
- operator/position/condition context.

A sequence is treated as complete only when its active elements and parent totals reconcile within the application's capture tolerance.

---

## 6.15 Capture Recurring Tasks

When timing recurring work:

1. Select the owning role.
2. Select the exact **Recurring / Side Task**.
3. Confirm operator/position.
4. Capture one complete occurrence of the defined task.
5. Repeat enough observations to characterize normal task duration.
6. Confirm the recurrence model independently from the stopwatch sample.

Do not use the core paced-role timer for a refill or side task if that task has already been modeled as a recurring task. Doing so can double-count labor.

---

## 6.16 Classify Abnormal Conditions

Available condition classifications include:

- Normal
- Material difficulty
- Misfeed
- Operator interruption
- Other exception

Classify the condition based on what actually occurred.

Do not automatically exclude an abnormal observation. The observation may be useful evidence of process risk even if it is not appropriate for the normal timing standard.

---

## 6.17 Review Recent Samples During Collection

Use the Recent Samples table to catch obvious field-entry errors while the context is still known.

Check:

- correct role/task,
- correct position,
- correct condition,
- plausible duration,
- accidental tap/double tap,
- sequence completion.

When a sample is clearly a capture error, document the reason during review rather than silently modifying the raw observed duration.

---

# 7. Study Review and Validation

## 7.1 Review Readiness

Open **Review** and inspect the Study Readiness & Data Quality section.

Current heuristic targets include:

- 15+ Normal observations for each paced role,
- 8+ Normal observations for each recurring task,
- valid recurrence definition for every included recurring task,
- usable measurement for every included paced role,
- sequence reconciliation where sequences are used,
- review of unresolved strong/extreme timing outliers.

These are application heuristics for data readiness. They do not prove that the study is statistically representative of every production condition.

---

## 7.2 Review Sample Statistics

For each role/task inspect:

- N
- mean
- median
- sample standard deviation
- coefficient of variation
- P90
- operator-weighted mean when operators are represented
- between-operator variation

Large differences between mean and median, high CV, or a P90 substantially above the mean warrant investigation even when no sample is automatically flagged.

---

## 7.3 Review Timing Outliers

Open the Data Accuracy / outlier review.

For each candidate decide whether it is:

- a valid observation to **Keep**,
- a valid observation requiring **Reclassification**,
- a capture error to **Exclude**, or
- another invalid sample requiring an explanatory note.

Outlier flags are advisory. Do not exclude solely because a sample is statistically unusual.

See **REF-AE-CTL-003** for the exact method and thresholds.

---

## 7.4 Review Sequence Reconciliation

For each sequenced role/task:

- confirm enough complete current-revision cycles exist,
- review incomplete cycles,
- review parent elapsed versus element-stack reconciliation,
- confirm wait/non-labor elements were classified correctly,
- investigate reconciliation states above the normal threshold.

The parent sample remains authoritative; element detail is used to explain and reconcile it.

---

## 7.5 Review Operator and Position Variation

Use Operator / Position Timing to check whether variation is:

- common to the process,
- associated with a particular physical position,
- associated with an operator alias,
- caused by uneven work content or access.

The application calculates an operator-weighted mean as the mean of each represented operator's own mean, so each represented operator contributes equally to that particular statistic regardless of sample count.

---

## 7.6 Review Process Timing and Capacity

For a paced role:

```text
Effective work sec/reference unit/position
= mean action sec × events/reference unit / positions
```

```text
Role capacity (reference units/min)
= 60 / effective work sec/reference unit/position
```

For sequenced work, elapsed-cycle capacity can impose a separate physical constraint. The measured role constraint uses the tighter applicable labor/elapsed limitation.

Do not assume the highest comparison rate shown in a diagnostic is a safe operating target. Review equipment and process limits separately.

---

## 7.7 Review Yamazumi / Standard Work Balance

Use the Yamazumi view to compare roles on one scale and inspect work-element or recurring-task composition.

Review both available concepts where applicable:

- **Labor Balance** — labor content that consumes worker capacity.
- **Cycle Composition** — elapsed sequence content, including wait/non-labor components.

Use the selected pace line as a comparison reference. Do not interpret the chart as an automatic work-assignment optimizer.

See **REF-AE-CTL-002** for detailed interpretation.

---

## 7.8 Review Shared Support Workload

For a support pool:

```text
Support load %
= Σ(task mean sec × events/hr)
  / (positions × 3600 × design efficiency)
  × 100
```

A support load below 100% does not guarantee that every request can be serviced at the moment it occurs. Consider travel, concurrency, skill restrictions, and request clustering.

---

## 7.9 Review Staffing Outputs

The application reports at least three staffing concepts:

### Theoretical Minimum

```text
N_theoretical = CEILING(total labor / reference interval)
```

Assumes 100% utilization and perfect rebalance.

### Design Minimum

```text
N_design = CEILING(total labor / (reference interval × design efficiency))
```

Adds the selected design-efficiency allowance but still aggregates labor before rounding.

### Constraint-Aware Design

Dedicated paced roles are calculated/rounded individually; shared support work is pooled before rounding. Other unmodeled fixed labor is handled separately where applicable.

Use Constraint-Aware Design as a more structure-aware planning result, but still verify the actual physical work assignment.

---

# 8. Finalize and Save

## 8.1 Final Review Checklist

Before saving a baseline:

- [ ] Study identity reflects the observed run.
- [ ] Primary reference rate is valid.
- [ ] Role relationships are correct.
- [ ] Material quantities are not confused with timing frequency.
- [ ] Current positions and worker fractions are accurate.
- [ ] Paced roles have representative timing coverage.
- [ ] Recurring tasks have representative timing and valid recurrence.
- [ ] Strong/extreme outlier candidates have decisions.
- [ ] Sequence mismatches are resolved or documented.
- [ ] Operator/position variation has been reviewed.
- [ ] Staffing/capacity results are physically plausible.
- [ ] Safety, quality, ergonomic, equipment, and skill constraints have been considered outside the mathematical model.

## 8.2 Save Study

Select **Data / Export → SAVE STUDY**.

A normal repository save writes:

```text
CT-RECORD-ID__STUDY.json
CT-RECORD-ID__REPORT.json
CT-RECORD-ID__TIMING-SAMPLES.xlsx   (when timing samples exist)
CT-RECORD-ID__MANIFEST.json
```

The Study JSON is the authoritative continuation/audit record. The Report JSON is the flattened downstream analysis package.

> **Warning:** Do not treat Report JSON as the authoritative resume file.

---

# 9. Starting the Next Study

## 9.1 Start New Study — Keep Line Setup

Use this function when the physical line/configuration is being reused for a new run.

It creates a **new Study ID** and preserves configured line structure, including roles, position counts, Areas, support-task definitions, machine/pitch setup, design efficiency, plant/line/equipment, and observer defaults.

It clears run-specific/captured information including operators, timing samples, belt-speed trials, placement-window trials, machine-rate samples, job/run identity, total headcount, notes, and report sequence. The next report begins again at `R001`.

Save the prior study first.

## 9.2 Full Reset — Clear Everything

Use **FULL RESET** only when a completely blank tool is intended.

It removes loaded mapping standards, roles, positions, support tasks, machine setup, operators, and captured data from the active browser study and replaces the local autosave.

The application requires confirmation because the operation is destructive to the active local state.

---

# 10. Records and Retention

Retain the generated Study Repository package according to local engineering records practice. At minimum, keep the Study JSON and its manifest together with any downstream report/sample files that were generated for that record.

Do not rely on browser autosave as the only record of a completed study.

---

## Related Documents

- **QS-AE-CTL-001** — Quick Start
- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **REF-AE-CTL-002** — Yamazumi & Standard-Work Analysis
- **REF-AE-CTL-003** — Timing Outlier Review
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
- **DATA-AE-CTL-001** — Data Handling & Study Repository
