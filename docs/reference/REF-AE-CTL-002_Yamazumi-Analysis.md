---
documentId: REF-AE-CTL-002
title: Cycle Time & Labor Study — Yamazumi & Standard-Work Analysis
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Yamazumi & Standard-Work Analysis

## 1. Purpose

The Cycle Time & Labor Study uses Yamazumi-style views to expose **where work exists, how that work is composed, and how the measured work compares with a selected production pace**.

The chart is intended to support engineering judgment about balance and standard work. It does **not** automatically redistribute work or prove that a proposed rebalance is physically achievable.

---

# 2. Two Different Questions

The application distinguishes two views that should not be confused.

## 2.1 Labor Balance

**Question:** How much labor capacity does this work consume?

Labor Balance emphasizes element/task time marked as labor and recurring workload assigned to the role/pool.

Use it for:

- labor-content comparison,
- role loading,
- line-balance opportunities,
- understanding where staffing demand is concentrated.

## 2.2 Cycle Composition

**Question:** What consumes elapsed cycle time, whether or not it is direct labor?

Cycle Composition retains elapsed sequence structure, including wait/idle or other non-labor elements.

Use it for:

- identifying waiting inside the cycle,
- distinguishing manual work from elapsed process time,
- understanding why a role can be physically constrained even when direct labor appears lower.

> **Key distinction:** Labor seconds and elapsed seconds can be different for the same parent cycle.

---

# 3. Sequence Data Model

A sequenced observation is stored as one parent cycle with child element detail.

```text
Parent cycle
├── Element 1
├── Element 2
├── Element 3
└── ...
```

The parent cycle remains authoritative for the existing timing/labor contract. Elements are used to explain and reconcile that parent observation.

This prevents one physical cycle from being accidentally counted as several independent process samples.

---

# 4. Labor-Included and Non-Labor Elements

Each active sequence element can contribute to labor or be excluded from labor content.

A non-labor or wait element can still contribute to parent elapsed time.

Conceptually:

```text
Parent elapsed time
= Σ all active element elapsed seconds
```

```text
Parent labor time
= Σ active element seconds where Include in Labor = true
```

This distinction allows a sequence to show, for example, a worker waiting for an indexed movement without pretending that the waiting period is productive manual work.

---

# 5. Current-Revision Rule

Sequence definitions carry identity and revision information.

The reconciled Yamazumi uses complete cycles belonging to the **current sequence revision**. Incomplete cycles or cycles captured under an obsolete sequence definition are not silently mixed into the current standard-work stack.

When the standard method changes materially, update the sequence definition and capture enough current-revision cycles to establish the new basis.

---

# 6. Selection of Sequence Cycles

For a sequenced paced role/task, the application:

1. finds current-revision cycles,
2. validates whether each cycle is complete,
3. prefers complete included **Normal** cycles,
4. falls back to complete non-excluded cycles only when no Normal complete cycles exist,
5. labels fallback use for review.

The same condition logic used by ordinary timing therefore remains visible in sequence analysis.

---

# 7. Sequence Completeness

A current-revision sequence cycle is considered complete only when the active sequence structure and parent totals agree.

The validation requires, in principle:

- every active element appears exactly once,
- no unexpected/archived element is present in the current active set,
- summed element elapsed time agrees with parent elapsed time,
- summed labor-included element time agrees with parent labor time.

The low-level capture validation tolerance is:

```text
Tolerance = MAX(0.005 sec, 0.25% of the compared parent total)
```

This tolerance handles small timer/rounding differences while still detecting missing or duplicated boundaries.

---

# 8. Element Statistics

For each active element, the application calculates timing statistics from the selected complete current-revision cycles.

For a paced role, the normalization factor is:

```text
F = process events/reference unit / current positions
```

An element's effective mean on the line-balance scale is:

```text
Element effective mean
= raw element mean × F
```

Likewise, parent labor and elapsed cycle values can be normalized to the same reference-unit/position basis.

This is why a role that performs multiple process events per finished unit or has multiple parallel positions can be meaningfully compared against a common reference pace.

---

# 9. Pace Reference

The chart can compare work against a selected pace basis such as:

- Primary Machine / Reference Rate,
- Belt-Derived Reference Rate,
- Placement Opportunity Window,
- Measured Bottleneck Rate,
- Fastest Available comparison signal.

For a selected rate `R_sel`:

```text
Selected pace interval
= 60 / R_sel
```

The pace line is a **comparison reference**. It does not alter the raw timing samples.

> **Important:** “Fastest Available” is a demanding comparison target, not necessarily a safe or achievable operating rate.

---

# 10. Interpreting Bar Height

For a paced role, a higher normalized stack means more time is required per reference unit at each position.

If the work stack approaches or exceeds the selected pace interval:

- recovery margin is reduced,
- the process may require more positions or a different work assignment,
- a sequence may contain excessive wait/elapsed content,
- the selected reference relationship may be incorrect,
- the selected comparison pace may be more demanding than the actual operating basis.

Investigate the cause before changing staffing.

---

# 11. Heat-Map Loading

The current visual uses escalating load colors (for example healthy, watch, tight, and over conditions) to make imbalance visible.

Treat color as a diagnostic priority, not as an approval decision.

A role can appear mathematically acceptable yet still be impractical due to:

- reach/travel,
- simultaneous tasks,
- machine interaction,
- ergonomic limits,
- skill restrictions,
- quality requirements,
- upstream/downstream coupling.

Conversely, a role can appear overloaded because the configured process relationship or current-position count is wrong.

---

# 12. Recurring Work in the Balance

Recurring side work assigned to a paced role contributes normalized labor to that role's labor demand.

Shared support roles are evaluated as pooled intermittent workload rather than as one action every machine cycle.

For a support pool:

```text
Support labor sec/hour
= Σ(task mean sec × task events/hour)
```

```text
Support load %
= support labor sec/hour
  / (current support positions × 3600 × design efficiency)
  × 100
```

The consolidated line-balance view can therefore compare paced roles and recurring support pools on a common visual scale while retaining different underlying workload models.

---

# 13. Sequence Reconciliation

In addition to the strict per-cycle completeness check, the application evaluates aggregate parent-versus-element reconciliation for the selected sequence basis.

The principal elapsed reconciliation percentage is based on the difference between the selected parent elapsed mean and the summed selected element means.

Conceptually:

```text
Elapsed delta %
= |element-sum mean - parent elapsed mean|
  / parent elapsed mean
  × 100
```

Current status thresholds are:

| Elapsed delta | Status |
| ---: | --- |
| `≤ 1%` | Reconciled |
| `> 1% to 3%` | Minor |
| `> 3% to 5%` | Review |
| `> 5%` | Mismatch |

If no valid complete selected cycles are available, the sequence remains incomplete for the current basis.

> **Interpretation:** The strict cycle-level check establishes whether an individual observation is structurally complete. The aggregate percentage is a second review layer for whether the selected element stack represents the parent cycle consistently across the sample set.

---

# 14. Parent Cycle Authority

When parent and element views disagree, investigate the sequence data rather than manually forcing the element stack to equal the desired total.

The parent observation remains the authoritative record of the captured cycle.

Potential causes of disagreement include:

- missed element boundary,
- duplicated boundary,
- sequence definition changed between observations,
- incomplete cycle retained in raw history,
- labor-inclusion setting inconsistent with the intended method,
- incorrect start/stop point,
- condition or exclusion differences.

---

# 15. Standard-Work Balancing Procedure

Use the following review sequence.

## Step 1 — Validate the basis

Confirm:

- selected reference rate,
- normalized process relationship,
- current positions,
- current sequence revision,
- sample condition basis.

## Step 2 — Validate completeness

Resolve or understand:

- incomplete cycles,
- reconciliation warnings,
- missing sequence coverage.

## Step 3 — Compare role totals

Identify roles that are:

- substantially below the selected pace,
- approaching the selected pace,
- exceeding the selected pace.

## Step 4 — Inspect element composition

For each role, determine which elements drive the total.

Look for:

- large repeated manual elements,
- duplicated motion,
- walking/reach burden represented in the cycle,
- inspection content,
- wait/non-labor time,
- recurring interruptions.

## Step 5 — Evaluate feasible transfers

Only then consider moving work between positions/roles.

A candidate transfer should be checked for:

- physical access,
- task sequence dependency,
- required skill,
- safety and ergonomic implications,
- material availability,
- quality ownership,
- machine timing.

## Step 6 — Re-study the revised method

A proposed rebalance becomes evidence-based only after the changed standard work is observed and timed again.

---

# 16. Labor Balance Example

Illustrative example only:

Suppose a role has:

- mean cycle action = `3.2 sec/event`,
- `2 events/reference unit`,
- `2 physical positions`.

```text
Effective work
= 3.2 × 2 / 2
= 3.2 sec/reference unit/position
```

At 15 reference units/min:

```text
Selected pace interval
= 60 / 15
= 4.0 sec/reference unit
```

Nominal reserve against that selected pace is:

```text
4.0 - 3.2 = 0.8 sec/reference unit/position
```

That 0.8 seconds is a mathematical comparison margin, not proof that the operation has 0.8 seconds of practically usable recovery every cycle.

---

# 17. Common Misinterpretations

### “The bar is below pace, so staffing is proven.”

No. The chart does not model every physical, ergonomic, or simultaneous-work constraint.

### “Wait should always be deleted.”

No. Wait may be real elapsed-cycle content. It should be classified correctly rather than hidden.

### “Element means can be added from unrelated samples.”

The tool reconciles elements through complete parent cycles/current sequence revision to avoid constructing a standard from incompatible fragments.

### “The Yamazumi automatically finds the optimal rebalance.”

No. It exposes measured imbalance. Work reassignment remains an engineering design decision.

### “Support work should be compared directly to every machine cycle.”

Not when it is intermittent. Shared support is modeled by recurrence and pooled labor demand.

---

# 18. Review Checklist

Before using the Yamazumi for a recommendation:

- [ ] Process-event relationships are correct.
- [ ] Position counts reflect the actual line.
- [ ] Selected pace is understood.
- [ ] Sequence revision is current.
- [ ] Complete Normal cycles are available, or fallback use is explicitly understood.
- [ ] Reconciliation is acceptable or reviewed.
- [ ] Labor/non-labor flags match the intended standard.
- [ ] Recurring side work is represented once, not double-counted.
- [ ] Operator/position variation has been checked.
- [ ] Any proposed work transfer is physically feasible.
- [ ] The revised method will be revalidated after implementation.

---

## Related Documents

- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **REF-AE-CTL-003** — Timing Outlier Review
- **WI-AE-CTL-001** — Study Procedure
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
