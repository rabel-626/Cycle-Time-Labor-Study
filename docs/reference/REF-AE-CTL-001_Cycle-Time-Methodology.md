---
documentId: REF-AE-CTL-001
title: Cycle Time & Labor Study — Methodology Reference
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Cycle Time & Labor Study — Methodology Reference

## 1. Purpose

This reference describes the principal calculation methods used by the **Abel Engineering Cycle Time & Labor Study v1.45.1**.

It is intended to make the study model auditable: a reviewer should be able to distinguish what was directly observed, what was entered as a process relationship, what was normalized, and what was calculated as a planning result.

> **Engineering principle:** Stopwatch time, production frequency, material quantity, staffing allowance, and physical process opportunity are separate concepts. The tool combines them only through explicitly defined relationships.

---

# 2. Calculation Hierarchy

At a high level:

```text
Raw timing observations
        ↓
Selected timing basis
        ↓
Mean / variation / P90
        ↓
Process events per reference unit
        ↓
Labor seconds per reference unit
        ↓
Reference throughput / interval
        ↓
Capacity, workload, and staffing calculations
```

Recurring work follows a parallel path:

```text
Raw recurring-task observations
        ↓
Selected task mean
        ↓
Task recurrence model
        ↓
Events per hour / events per reference unit
        ↓
Labor sec/hour and labor sec/reference unit
        ↓
Paced-role side work or pooled support load
```

---

# 3. Reference Unit and Reference Throughput

The **reference unit** is the common finished-unit basis used to compare processes with different frequencies.

Let:

| Symbol | Meaning |
| --- | --- |
| `R_ref` | Primary reference throughput, reference units/min |
| `T_ref` | Reference interval, sec/reference unit |

```text
T_ref = 60 / R_ref
```

Example:

```text
R_ref = 10 units/min
T_ref = 60 / 10 = 6 sec/unit
```

The current application treats the direct **Primary Observed Output Rate** as the staffing/reference rate. Alternative rate signals are diagnostics or comparison bases unless the user deliberately updates the primary rate.

---

# 4. Sample Selection Basis

The application does not average every stored observation indiscriminately.

For an applicable timing group:

1. Excluded observations are removed from the selected calculation basis.
2. Positive-duration **Normal** observations are preferred.
3. If at least one Normal observation exists, selected statistics use the Normal observations.
4. If no Normal observation exists, the application falls back to other non-excluded observations and identifies that fallback for review.

For an enabled standard-work sequence, only **complete cycles from the current sequence revision** are eligible for the reconciled sequence timing basis.

> **Important:** A fallback basis allows analysis to remain visible when normal data is unavailable; it is not equivalent to a fully established normal-production baseline.

---

# 5. Descriptive Timing Statistics

Let the selected positive-duration observations be:

```text
x1, x2, ..., xn
```

## 5.1 Mean

```text
x̄ = (Σ xi) / n
```

The mean is used as the principal labor-content timing value for paced roles and recurring tasks.

## 5.2 Median

After sorting the observations:

- odd `n`: middle value,
- even `n`: average of the two middle values.

The median is particularly useful as a robust central reference for the outlier-review method.

## 5.3 Sample Standard Deviation

The current application uses **sample standard deviation**:

```text
s = sqrt( Σ(xi - x̄)² / (n - 1) )
```

For fewer than two values, the application returns zero.

## 5.4 Coefficient of Variation

```text
CV = s / x̄
```

A higher CV indicates greater relative timing variation, but there is no universal CV threshold that proves a process is acceptable or unacceptable.

## 5.5 P90

The current application uses a **nearest-rank** 90th percentile:

```text
rank = CEILING(0.90 × n)
P90 = sorted observation at that rank
```

P90 is used in work-window/recovery analysis to compare a high-side observed work duration with available physical or selected-pace opportunity.

Because nearest-rank P90 changes in steps for small sample sets, interpret it cautiously at low `n`.

---

# 6. Operator-Weighted Mean

For selected samples with operator aliases, the application first calculates each operator's own mean and then averages those operator means.

For `m` represented operators:

```text
Operator-weighted mean
= (operator mean1 + ... + operator meanm) / m
```

This gives each represented operator equal weight in this statistic, regardless of how many selected samples each operator contributed.

The application also calculates sample standard deviation across the operator means as a between-operator variation measure.

> **Interpretation:** The operator-weighted mean is useful for identifying sample-count imbalance, but it does not replace review of position, method, skill, and process differences.

---

# 7. Process-Event Normalization

A process role can be defined in either direction.

## 7.1 Events per reference unit

If the entry directly states process events per reference unit:

```text
E_ref = entered events/reference unit
```

## 7.2 Reference units per process event

If one process event serves `U_event` reference units:

```text
E_ref = 1 / U_event
```

This normalized `E_ref` is used consistently in labor, material, capacity, and reporting calculations.

---

# 8. Component Consumption

Material quantity is separated from process timing frequency.

Let:

| Symbol | Meaning |
| --- | --- |
| `E_ref` | Process events/reference unit |
| `C_event` | Components/process event |
| `C_ref` | Components/reference unit |

```text
C_ref = E_ref × C_event
```

At a reference rate `R_ref`:

```text
Component demand/min = R_ref × C_ref
```

This prevents one observed process event that handles multiple physical components from incorrectly multiplying the measured stopwatch duration.

---

# 9. Paced Role Labor

For a paced role with selected mean action time `t_mean`:

```text
Core paced labor sec/reference unit
= t_mean × E_ref
```

If recurring side tasks are assigned to the paced role, their normalized labor is added to the role's labor content.

Conceptually:

```text
Role labor/reference unit
= core paced labor/reference unit
+ assigned recurring-task labor/reference unit
```

The number of current physical positions does **not** divide total labor content. Positions are a capacity/resource constraint; the same total work still has to be performed.

---

# 10. Effective Work Per Position and Process Capacity

For a simple one-shot paced role:

```text
Effective work sec/reference unit/position
= t_mean × E_ref / P
```

where `P` is the number of current physical positions.

The corresponding simple process capacity is:

```text
Capacity (reference units/min)
= 60 / effective work sec/reference unit/position
```

This is useful for parallel positions performing equivalent demand.

For a sequenced role, the application can also use the complete parent **elapsed** cycle as a physical-capacity constraint. The measured role constraint uses the tighter applicable elapsed or labor constraint.

---

# 11. Measured Bottleneck Method

The current measured-bottleneck calculation evaluates active paced roles and excludes support pools from the paced bottleneck.

For each paced role, the application considers:

1. **variable labor demand per reference unit**, including normalized paced work and applicable recurring side work; and
2. **fixed auxiliary seconds/hour** where recurring work is defined on an Events/Hour basis.

Available labor seconds/hour are reduced by fixed Events/Hour burden:

```text
Available role labor sec/hour
= positions × 3600 - fixed auxiliary sec/hour
```

Then:

```text
Labor-capacity rate
= available role labor sec/hour
  / (variable labor sec/reference unit × 60)
```

Where a valid sequence reconciliation exists, elapsed-cycle capacity is also evaluated:

```text
Elapsed-capacity rate
= 60 / effective parent elapsed sec/reference unit/position
```

The role capacity is the tighter applicable constraint, and the **Measured Bottleneck Rate** is the minimum capacity across measured paced roles.

> **Engineering note:** This is a model of configured/measured roles. An omitted role, incorrect position count, invalid recurrence, equipment limitation, or physical interaction not represented in the model can make the calculated bottleneck incomplete.

---

# 12. Recurring Task Frequency

A recurring task can be modeled in three principal ways.

## 12.1 Consumption-driven

Let:

- `C_ref` = component demand/reference unit,
- `Q_service` = components replenished/handled per task event.

```text
Task events/reference unit
= C_ref / Q_service
```

At throughput `R_ref`:

```text
Task events/hour
= task events/reference unit × R_ref × 60
```

## 12.2 Reference-unit recurrence

The user may directly enter events/reference unit or the reciprocal reference units/task event. The tool normalizes the result to events/reference unit.

## 12.3 Events/hour

For a fixed event frequency `F_hr`:

```text
Task events/reference unit
= F_hr / (R_ref × 60)
```

---

# 13. Recurring Task Labor

For selected mean task time `t_task`:

```text
Task labor sec/hour
= t_task × task events/hour
```

```text
Task labor sec/reference unit
= t_task × task events/reference unit
```

For a shared support pool, task workloads are summed before evaluating pooled utilization and design FTE.

---

# 14. Shared Support Pool Workload

Let:

- `L_support_hr` = summed modeled support labor sec/hour,
- `P_support` = current support positions,
- `η` = design efficiency as a decimal.

```text
Support pool load %
= L_support_hr / (P_support × 3600 × η) × 100
```

Design support FTE is:

```text
Support design FTE
= L_support_hr / (3600 × η)
```

The application pools the intermittent tasks assigned to the support role before evaluating support design positions.

> **Limitation:** Average pooled load does not model the full probability of simultaneous requests, travel conflicts, or skill restrictions.

---

# 15. Material Replenishment Frequency

For a component/refill relationship:

```text
Refill events/hour
= R_ref × C_ref × 60 / Q_refill
```

where `Q_refill` is the quantity supplied per refill/service event.

This is a demand estimate from configured consumption relationships; it does not prove that a material handler can physically perform all required refills within the available travel and handling time.

---

# 16. Belt-Derived Reference Rate

Let:

- `V_belt` = conveyor linear speed,
- `D_pitch` = physical repeating pitch distance in compatible units,
- `U_pitch` = reference units per belt pitch.

```text
Physical belt pitches/min = V_belt / D_pitch
```

```text
Belt-derived reference rate
= physical belt pitches/min × U_pitch
```

This separates physical pitch events from cases where multiple reference units occupy one pitch.

### Synchronized belt speed

```text
Expected synchronized belt speed
= (Primary reference rate / U_pitch) × D_pitch
```

If actual VFD frequency and measured belt speed are available:

```text
Estimated synchronized Hz
= actual Hz × expected synchronized speed / measured speed
```

A target feed ratio can then be applied as a calibration aid.

> **Caution:** VFD recommendations are proportional calibration estimates, not substitutes for machine/vendor drive limits or controls engineering review.

---

# 17. Placement Opportunity Window

For an observed usable window:

- `W` = measured window seconds,
- `U_w` = reference units represented by the window,
- `K` = optional window factor.

```text
Opportunity interval
= W / U_w / K
```

Equivalent rate:

```text
Opportunity rate = 60 / opportunity interval
```

The opportunity window is a physical/process diagnostic. It does not automatically replace the primary staffing/reference throughput.

---

# 18. Work Window and Recovery

When a physical work zone and conveyor speed are used:

```text
Physical work-window sec
= work-zone length / conveyor speed × 60
```

For one-shot timing:

```text
P90 effective work
= P90 action sec × E_ref / positions
```

For a sequenced role, the current implementation uses complete-cycle parent elapsed P90 for the physical-cycle comparison.

Reserve is evaluated as:

```text
Window reserve
= available window sec - P90 effective work sec
```

The tool can also compare work against a selected pace interval. Physical work window and selected pace are separate constraints and should not be conflated.

---

# 19. Total Modeled Labor

Conceptually, the line-level model is:

```text
Total labor/reference unit
= paced-role labor
+ shared support labor
+ modeled fixed labor terms
```

The current v1.45.1 code retains a `rework` term in the aggregate structure/report for compatibility, but the current aggregate `reworkLabor()` implementation evaluates to **zero**. Therefore, do not assume observed rework event duration is automatically converted into labor seconds/reference unit in the current staffing result.

> **Important:** If known recurring labor is not represented by a timed role/task and is entered as **Unmodeled Fixed Labor sec/reference unit**, remove that fixed allowance if the work is later modeled explicitly, or it will be double-counted.

---

# 20. Current Crew Load

Let:

- `L_total` = total modeled labor sec/reference unit,
- `T_ref` = reference interval,
- `N_current` = current configured worker equivalent.

```text
Current crew load
= L_total / (T_ref × N_current)
```

Configured worker equivalent is based on the applicable current positions and worker fractions included in headcount.

A mathematical load below 100% does not prove the current staffing pattern is feasible because labor may be constrained by role ownership, timing, travel, or simultaneous demand.

---

# 21. Staffing Calculations

## 21.1 Theoretical Minimum

```text
N_theoretical
= CEILING(L_total / T_ref)
```

Assumptions:

- 100% labor utilization,
- perfect rebalance,
- no role ownership constraints,
- no practical recovery allowance.

## 21.2 Aggregate Design Minimum

```text
N_design
= CEILING(L_total / (T_ref × η))
```

This applies the selected design efficiency but still assumes aggregate rebalance before rounding.

## 21.3 Constraint-Aware Design

The current application separately calculates/rounds:

- each dedicated paced role,
- each shared support pool after pooling its tasks,
- applicable other fixed labor.

Conceptually:

```text
N_constraint-aware
= Σ dedicated paced-role design positions
+ Σ pooled support design positions
+ other separately rounded design positions
```

This retains more of the role structure than the aggregate design minimum.

> **Interpretation:** A higher constraint-aware result is not an error; it often represents losses in theoretical pooling/rebalancing caused by dedicated roles or separated support pools.

---

# 22. Quick Scenario Model

The Scenario model is intentionally an **aggregate sensitivity calculation**.

It first estimates an average paced seconds/event from the current model:

```text
Average paced sec/event
= current paced labor sec/reference unit
  / current normalized paced events/reference unit
```

Scenario paced labor is then approximated as:

```text
Scenario paced labor
= average paced sec/event × scenario work quantity
```

Scenario interval:

```text
Scenario interval
= 60 / scenario reference throughput
```

Scenario total labor includes the scaled core paced work plus baseline recurring/support/fixed components and any user-entered additional fixed labor.

Scenario design staffing:

```text
Scenario design workers
= CEILING(scenario total labor
  / [scenario interval × scenario efficiency])
```

> **Limitation:** This model assumes added/removed paced work resembles the average paced work already measured. It is not a component-specific future-state model. Detailed component/material what-if analysis belongs in the downstream Material Flow & Takt Planner.

---

# 23. Readiness Heuristics

Current v1.45.1 quality checks use the following notable thresholds:

| Check | Current behavior |
| --- | --- |
| Primary rate missing | Blocker |
| No process roles | Blocker |
| Paced role with no usable timing | Blocker |
| Paced role with fewer than 15 Normal selected observations | Warning |
| Recurring task with no usable timing | Blocker |
| Recurring task with fewer than 8 Normal selected observations | Warning |
| Incomplete recurrence definition | Blocker |
| Strong/extreme unresolved outlier | Warning |
| Multi-position role with fewer than 2 represented operator aliases | Warning |
| Area missing | Warning |
| Significant sequence mismatch | Review warning or blocker depending on severity |

These thresholds are **application heuristics**, not universal industrial-engineering standards. The required study depth can be greater depending on process variation, risk, and intended use.

---

# 24. Interpretation Principles

Use the model in this order:

1. **Validate what was observed.**
2. **Validate what one observation represents.**
3. **Validate process frequency relative to the reference unit.**
4. **Validate recurrence assumptions.**
5. **Review variation and outliers.**
6. **Review physical process opportunity/capacity.**
7. **Then interpret labor and staffing outputs.**

A precise equation does not correct an incorrect process definition.

---

## Related Documents

- **WI-AE-CTL-001** — Study Procedure
- **REF-AE-CTL-002** — Yamazumi & Standard-Work Analysis
- **REF-AE-CTL-003** — Timing Outlier Review
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
