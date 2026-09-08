---
documentId: QS-AE-CTL-001
title: Cycle Time & Labor Study — Quick Start
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Cycle Time & Labor Study — Quick Start

The **Abel Engineering Cycle Time & Labor Study** is a browser-based industrial-engineering tool for structuring a production study, capturing process and recurring-task timing, analyzing labor content and process capacity, reviewing operator and timing variation, and producing a controlled study package for downstream analysis.

> **Scope:** Use this tool for cycle time, standard-work timing, labor content, recurring support work, process capacity, and line-balance analysis. Use the **Downtime Tracker** for line downtime and operating-loss studies.

---

## 1. Recommended Workflow

| Step | Action | Result |
| --- | --- | --- |
| 1 | Connect or open the **Study Repository** | Controlled save/open location is available |
| 2 | Complete **Study** and **Production Reference** setup | The study has identity and a common reference-unit pace |
| 3 | Define **Areas, Process Roles, Positions, and relationships** | Work is normalized to the finished reference unit |
| 4 | Define recurring/side tasks and optional standard-work sequences | Intermittent labor and element-level work can be modeled |
| 5 | Capture representative timing observations | Raw cycle/task observations are recorded with context |
| 6 | Review data quality, outliers, Yamazumi, capacity, and staffing | The baseline is checked before use |
| 7 | **SAVE STUDY** | Study, report, sample workbook, and manifest are written to the repository |

---

## 2. Before You Begin

Have the following information available where applicable:

- Plant, line, equipment, job/SKU, shift, study date, and observer.
- The **reference unit** used to describe finished throughput, such as carton, case, assembly, or another defined production unit.
- A validated **Primary Observed Output Rate** in reference units per minute.
- The process roles that perform repeated work and the number of physical positions for each role.
- The relationship between each process event and the finished reference unit.
- Component quantities, refill quantities, or recurrence frequencies for recurring work when those functions are being modeled.
- Non-sensitive operator aliases such as `OP-01`, `OP-02`, and so on.

> **Important:** The Primary Observed Output Rate is the staffing/reference basis used by the study. Belt-derived, placement-opportunity, and measured-bottleneck rates are comparison signals; they do not silently replace the primary staffing rate.

---

## 3. Connect the Study Repository

Open **Data / Export** and select **CONNECT / RECONNECT REPOSITORY**.

The current repository workflow requires desktop **Microsoft Edge or Google Chrome** with browser support for folder access. Select the approved repository root named exactly:

- `Engineering Study Hub - Study Repository`, or
- `Study Repository`

Cycle Time studies are stored under:

```text
Study Repository/
└── 01_CYCLE_TIME/
    └── PLANT/
        └── LINE/
            └── YYYY/
                └── YYYY-MM/
                    └── CT-RECORD-ID/
```

The browser may require folder permission to be granted again after a restart or security-policy change.

---

## 4. Establish the Production Reference

The study compares work against a common finished-unit pace.

### Reference interval

```text
T_ref = 60 / R_ref
```

| Variable | Meaning |
| --- | --- |
| `T_ref` | Reference interval, seconds per finished reference unit |
| `R_ref` | Primary reference throughput, reference units per minute |

**Example:** At 12 cartons/minute, the reference interval is 5 seconds/carton.

```text
T_ref = 60 / 12 = 5.00 sec/carton
```

Use the machine-rate, conveyor/belt, or placement-opportunity measurements as validation/context when useful, but confirm the direct primary rate before relying on staffing results.

---

## 5. Define the Process Structure

The study hierarchy is:

```text
Study
  → Area
    → Process Role
      → Core Work
      → Assigned Recurring / Side Tasks
      → Optional Sequence Elements
      → Physical Position
      → Operator Observation
```

For each repeated **Process Role**, define at minimum:

- Area.
- Role/category and descriptive name.
- Work model: paced/repeating, recurring/support, or reference-only as appropriate.
- Current physical positions.
- Worker fraction if a position represents less than one full person.
- Relationship to the finished reference unit.
- Component quantity per process event when material consumption is relevant.
- Whether the role is included in routine labor/headcount calculations.

### Normalize process frequency

A role may be entered in either direction. The application normalizes both to **process events per reference unit**.

```text
If entry = events per reference unit:
E_ref = entered value

If entry = reference units per process event:
E_ref = 1 / entered value
```

### Core paced labor

```text
L_role = t_mean × E_ref
```

| Variable | Meaning |
| --- | --- |
| `L_role` | Core paced labor seconds per reference unit |
| `t_mean` | Selected mean seconds per process event |
| `E_ref` | Normalized process events per reference unit |

---

## 6. Configure Recurring / Side Tasks

Use recurring tasks for lower-frequency work such as:

- replenishing hand stock,
- grabbing a new stack,
- removing empties,
- periodic material handling,
- routine side work assigned to a paced role, or
- shared support work performed by a support pool.

A recurring task must have both:

1. a measured task duration, and
2. a valid recurrence model.

Supported recurrence bases include:

- consumption-driven frequency,
- events per reference unit / reference units per event,
- fixed events per hour.

For a consumption-driven task:

```text
Task events/reference unit
= component demand/reference unit ÷ quantity per service event
```

---

## 7. Optional: Define a Standard-Work Sequence

Use a sequence when one repeated cycle should be broken into ordered work elements.

A sequence observation remains **one parent cycle**. The individual elements are child detail under that cycle; they are not treated as independent ordinary cycle samples.

Sequence capture supports:

- **Continuous Cycles** — move directly from one completed cycle into the next.
- **Single Cycle** — capture one complete cycle and stop.
- Element boundary timing with one action.
- Undo of the last boundary.
- Stop after the current cycle.
- Cancel/discard of the current incomplete cycle.
- Focused full-screen capture for field use.

Elements may be marked as labor or non-labor. Wait/idle elements can remain part of elapsed cycle time without being added to labor content.

---

## 8. Capture Timing Samples

On **Capture**:

1. Select an **Area** if filtering is useful.
2. Select the exact **Process / Role**.
3. Select a **Recurring / Side Task** only when timing that task; otherwise leave the core-role selection active.
4. Choose the actual **Physical Position**.
5. enter a consistent **Operator Alias**.
6. Confirm the **Sample Condition**.
7. Start the capture window or sequence.

### Sample conditions

Use **Normal** for representative production. Current exception choices include:

- Material difficulty
- Misfeed
- Operator interruption
- Other exception

The timing model prefers included **Normal** observations when at least one Normal observation exists. If none exist, it falls back to other non-excluded observations and labels that fallback for review.

> **Good practice:** Capture normal production first. Classify abnormal conditions when they occur instead of deleting them merely because they differ from the expected cycle.

---

## 9. Minimum Review Targets

The application uses the following **readiness heuristics**. They are review thresholds, not universal statistical guarantees.

| Item | Current heuristic |
| --- | ---: |
| Paced role | **15+ Normal** selected samples/complete cycles |
| Recurring task | **8+ Normal** selected samples/complete cycles |
| Outlier screening | Begins with **5** compatible baseline observations |
| Multi-position role | Prefer observations representing at least **2 operator aliases** |
| Sequence | Current revision must reconcile and contain complete cycles |
| Recurring task | Valid recurrence definition required |
| Staffing model | Valid Primary Observed Output Rate required |

Study readiness is presented as **Developing**, **Review Warnings**, or **Baseline Ready** based on the application's validation checks.

---

## 10. Review the Study

Before treating a study as a baseline, inspect:

- Study Readiness & Data Quality.
- Role timing summary.
- Operator / position timing.
- Full sample history.
- Timing outlier candidates and review decisions.
- Sequence reconciliation and incomplete cycles.
- Process timing versus reference pace.
- Operation work-window/recovery analysis when configured.
- Yamazumi / Standard Work Balance.
- Shared support workload.
- Labor content and staffing outputs.

### Core staffing equations

The 100%-utilization theoretical minimum is:

```text
N_theoretical = CEILING(L_total / T_ref)
```

The efficiency-adjusted aggregate design minimum is:

```text
N_design = CEILING(L_total / (T_ref × η))
```

| Variable | Meaning |
| --- | --- |
| `L_total` | Total modeled labor seconds/reference unit |
| `T_ref` | Reference interval seconds/reference unit |
| `η` | Design efficiency expressed as a decimal |

The application also reports a **Constraint-Aware Design** that rounds dedicated paced roles individually and pools shared support work before rounding. This can be more realistic than assuming all work can be perfectly rebalanced across every person.

---

## 11. Save the Study

Use **Data / Export → SAVE STUDY**.

A normal repository save creates a verified record package containing:

| File | Purpose |
| --- | --- |
| `CT-…__STUDY.json` | Authoritative full-fidelity study/resume record |
| `CT-…__REPORT.json` | Flattened downstream analysis package |
| `CT-…__TIMING-SAMPLES.xlsx` | Timing sample workbook when samples exist |
| `CT-…__MANIFEST.json` | Package manifest and output inventory |

The application verifies JSON writes and records hashes for the Study and Report outputs. If the selected repository is inside a OneDrive-synchronized location, cloud/SharePoint synchronization is performed by **OneDrive**, not by the Cycle Time application itself.

> **Important:** Browser autosave is crash recovery, not the official completed-study record. Use **SAVE STUDY** before ending a study session, switching devices, clearing browser data, or making a major change.

---

## 12. Quick Finalization Checklist

Before using results for a staffing or process decision, confirm:

- [ ] Study identity is complete.
- [ ] Primary Observed Output Rate is validated.
- [ ] Role-to-reference-unit relationships are correct.
- [ ] Positions and worker fractions reflect the actual process.
- [ ] Normal samples are representative of the operating condition being studied.
- [ ] Recurring task frequencies are complete.
- [ ] Strong/extreme outlier candidates have been reviewed.
- [ ] Sequence mismatches/incomplete cycles have been resolved or understood.
- [ ] Operator/position variation has been considered.
- [ ] Yamazumi and process-capacity results are physically plausible.
- [ ] Study is saved to the Study Repository.
- [ ] Engineering, safety, ergonomic, quality, and equipment constraints are reviewed separately where applicable.

---

## Related Documents

- **WI-AE-CTL-001** — Cycle Time & Labor Study — Study Procedure
- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **REF-AE-CTL-002** — Yamazumi & Standard-Work Analysis
- **REF-AE-CTL-003** — Timing Outlier Review
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
- **DATA-AE-CTL-001** — Data Handling & Study Repository
