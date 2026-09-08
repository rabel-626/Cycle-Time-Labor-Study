---
documentId: REF-AE-CTL-003
title: Cycle Time & Labor Study — Timing Outlier Review
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Timing Outlier Review

## 1. Purpose

The timing outlier system is designed to identify **likely capture anomalies or statistically unusual observations that deserve human review**.

It is intentionally not an automatic data-deletion mechanism.

> **Core rule:** Unusual does not mean invalid. A long or short cycle may be an accurate observation of the process.

---

# 2. What the Outlier System Is Intended to Find

Examples include:

- missed stop/boundary clicks,
- timer started late,
- timer stopped early,
- wrong role/task timed,
- accidental tap/double tap,
- unusually long or short observations that may reflect a real process condition,
- abnormal sequence element timing hidden inside an otherwise plausible parent cycle.

The reviewer decides whether a candidate is a valid observation, a condition-classification issue, or a capture error.

---

# 3. Like-for-Like Comparison Groups

Samples are not compared indiscriminately across the entire study.

The current application groups observations by their timing owner and capture method.

Conceptually:

```text
Group = Role or Recurring Task
      + Single-timer or Sequential capture
      + Sequence ID/revision when sequential
```

This prevents, for example, a 30-second refill task from being statistically compared with a 2-second placement cycle merely because both exist in the same study.

---

# 4. Baseline Eligibility

The comparison baseline uses observations that are:

- in the same timing group,
- not excluded,
- classified **Normal**,
- structurally complete when sequential,
- positive duration.

A candidate can therefore be reviewed against the normal behavior of the specific work it represents.

---

# 5. Minimum Sample Count

No robust outlier flag is produced until at least **5 compatible baseline observations** exist.

The method deliberately becomes more sensitive only when the group has at least 10 observations.

| Baseline N | Screening behavior |
| ---: | --- |
| `< 5` | No outlier flag |
| `5–9` | Extreme ratio screening only |
| `10+` | Extreme, Strong, and Watch screening |

This prevents small data sets from generating an excessive number of advisory flags.

---

# 6. Median Ratio

For candidate duration `x` and baseline median `M`:

```text
Ratio = x / M
```

Examples:

- `Ratio = 2.0` means the observation is twice the group median.
- `Ratio = 0.5` means the observation is half the group median.

The median is used because it is less sensitive than the mean to a small number of extreme observations.

---

# 7. Log-MAD Robust Score

The application also calculates a robust score on **log-transformed positive durations**.

Let:

```text
yi = ln(xi)
y  = ln(x)
```

The median absolute deviation of log durations is:

```text
MAD_log = median( |yi - median(y)| )
```

The current robust score is:

```text
z_robust
= 0.6745 × [ln(x) - median(ln(xi))] / MAD_log
```

The log transform treats multiplicative timing changes more symmetrically. For example, “twice as long” and “half as long” are more naturally compared on a ratio/log scale than on a raw-seconds difference scale.

If the log MAD is effectively zero, the robust-Z component is not used.

---

# 8. Current Severity Thresholds

## 8.1 Five to nine baseline observations

Only **Extreme** candidates are flagged:

```text
Ratio ≥ 3.0
or
Ratio ≤ 1/3
```

## 8.2 Ten or more baseline observations

| Severity | Ratio trigger | Robust-score trigger |
| --- | --- | --- |
| **Extreme** | `≥ 3.0` or `≤ 1/3` | Ratio rule alone is sufficient |
| **Strong** | `≥ 2.0` or `≤ 0.50` | `|z_robust| ≥ 6` **and** ratio `≥ 1.75` or `≤ 0.57` |
| **Watch** | `≥ 1.5` or `≤ 2/3` | `|z_robust| ≥ 3.5` **and** ratio `≥ 1.25` or `≤ 0.80` |

The ratio guards on robust-score triggers prevent a large robust score from flagging a duration that is still very close to the group median in practical terms.

---

# 9. Why the Application Uses More Than One Trigger

A simple ratio threshold is easy to understand but cannot adapt to the natural spread of different processes.

A robust log-MAD score adapts to the group's observed dispersion but can become unstable when the group is extremely tight.

The combined method therefore uses:

- clear ratio thresholds,
- a robust dispersion-aware score,
- minimum sample-count rules,
- additional ratio guards.

This is a **screening system**, not a formal process-capability test.

---

# 10. Sequential Timing Review

Sequential observations are checked at two levels.

## 10.1 Parent elapsed cycle

The complete parent elapsed duration is compared with compatible parent cycles.

## 10.2 Individual sequence elements

Each element duration is compared with the same element in compatible baseline cycles.

This can expose a situation where the parent cycle is not extreme but one boundary appears to have been missed or shifted.

Example:

```text
Parent cycle: normal-looking 9.8 sec
Element A: 1.2 sec
Element B: 7.0 sec  ← unusually long
Element C: 1.6 sec
```

A boundary error between A/B/C may be easier to detect at element level than from the parent total alone.

---

# 11. Review Actions

The current accuracy-review workflow preserves the observation and records a decision.

## 11.1 Keep — Valid Observation

Use when the timing is unusual but accurately represents what occurred.

Examples:

- worker had a legitimate difficult manipulation still considered normal,
- naturally long tail of a variable manual task,
- an unusual but correct short cycle.

Keeping a sample records the review decision without changing the observed duration.

## 11.2 Reclassify

Use when the time was captured correctly but the condition should not be Normal.

Possible conditions include:

- Material difficulty
- Misfeed
- Operator interruption
- Other exception

This preserves the observation while moving it out of the normal baseline when appropriate.

## 11.3 Exclude — Capture Error

Use when the observation itself is not a valid timing record.

Current capture-error reasons include:

- Missed stop / boundary click
- Started late
- Stopped early
- Wrong task / role timed
- Accidental tap / double tap
- Other capture error

The sample is retained in the study with an exclusion reason and review metadata.

## 11.4 Exclude — Other Invalid Sample

Use only when a validly captured observation must be excluded for another documented reason.

The current application requires a reviewer note for this action.

## 11.5 Reset Review

Clears the outlier-review decision metadata so the sample can be reviewed again.

---

# 12. Recommended Decision Logic

Use this sequence for each flagged observation.

```text
Was the timer/capture performed correctly?
        │
   ┌────┴────┐
   │         │
  No        Yes
   │         │
Exclude     Did an identifiable abnormal
capture     condition affect the sample?
error           │
           ┌────┴────┐
           │         │
          Yes       No
           │         │
      Reclassify   Is the unusual value a
                  valid observation of the
                  defined normal process?
                       │
                  ┌────┴────┐
                  │         │
                 Yes       No / other issue
                  │         │
                 Keep      Document and use
                           appropriate exclusion
```

Do not begin with the question “Do I want this value in my average?” Begin with “What actually happened?”

---

# 13. Examples

## Example A — Missed stop click

Baseline median = 3.0 sec. Candidate = 10.2 sec.

```text
Ratio = 10.2 / 3.0 = 3.4
```

The candidate is Extreme. The observer recalls that the stop click was missed.

**Decision:** Exclude — Capture Error → Missed stop / boundary click.

## Example B — Genuine material difficulty

Baseline median = 4.0 sec. Candidate = 8.5 sec.

```text
Ratio = 8.5 / 4.0 = 2.125
```

The timing is real, but a warped component caused the event.

**Decision:** Reclassify → Material difficulty.

## Example C — Valid long normal cycle

Baseline median = 5.0 sec. Candidate = 7.8 sec. The observation was captured correctly and no exception occurred.

The value may be Watch/Strong depending on group size and robust score.

**Decision:** Keep, if review confirms it is valid normal variation.

The purpose of the review is not to make the distribution look cleaner.

---

# 14. How Outlier Decisions Affect Statistics

Selected timing statistics use non-excluded observations and prefer Normal condition when Normal data exists.

Therefore:

- **Keep** leaves a valid Normal sample in the selected baseline when it is classified Normal.
- **Reclassify** can remove it from the Normal baseline while preserving it as an observed exception.
- **Exclude** removes it from selected calculations but preserves the raw record and reason.

This design keeps raw capture history and analytical selection separate.

---

# 15. Readiness Impact

The current study-quality assessment treats unresolved outliers differently by severity:

- unresolved **Strong/Extreme** candidates generate a warning,
- unresolved **Watch** candidates are informational,
- outliers never delete themselves automatically.

A study can therefore remain visibly incomplete for review without silently rewriting its source data.

---

# 16. What the Outlier System Does Not Do

It does not:

- determine whether a process condition is operationally acceptable,
- determine whether the operator performed the correct standard method,
- prove that a sample is wrong,
- replace process observation/context,
- establish a statistically required sample size,
- calculate process capability,
- remove outliers automatically,
- normalize away real process variation.

---

# 17. Review Checklist

For each flagged candidate:

- [ ] Confirm role/task and sequence revision.
- [ ] Confirm physical position and operator alias.
- [ ] Confirm sample condition.
- [ ] Check whether the start/stop/boundary was correct.
- [ ] For sequences, inspect the flagged element as well as the parent cycle.
- [ ] Compare with nearby observations/context if useful.
- [ ] Keep valid normal variation.
- [ ] Reclassify correctly captured exceptions.
- [ ] Exclude only with a defensible reason.
- [ ] Add notes where the reason is not self-evident.

---

## Related Documents

- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **REF-AE-CTL-002** — Yamazumi & Standard-Work Analysis
- **WI-AE-CTL-001** — Study Procedure
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
