---
documentId: LIM-AE-CTL-001
title: Cycle Time & Labor Study — Limitations & Engineering Boundaries
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Limitations & Engineering Boundaries

## 1. Purpose

This document defines the principal limitations of the **Abel Engineering Cycle Time & Labor Study** and the boundaries that should be considered before using its outputs for staffing, production planning, process redesign, or downstream analysis.

The tool is an engineering analysis aid. It is not a substitute for field validation or professional engineering judgment.

---

# 2. Intended Use

The tool is intended to support:

- observational time studies,
- repeated process-role timing,
- element-level standard-work timing,
- operator/position variation review,
- recurring/support workload modeling,
- labor-content normalization,
- process capacity comparisons,
- Yamazumi / balance analysis,
- staffing sensitivity and design estimates,
- material consumption/replenishment relationships,
- controlled study data capture and export.

---

# 3. Not Intended For

The application is not intended to function as:

- a machine safety system,
- safety PLC or interlock logic,
- a protective device,
- real-time process control,
- a machine-rate command system,
- an ergonomic risk-assessment replacement,
- a quality acceptance system,
- a payroll/timekeeping system,
- a formal statistical process-control package,
- a substitute for the Downtime Tracker for formal downtime/operating-loss analysis.

---

# 4. Observation Quality Limits

The output cannot be more reliable than the observations and process definitions entered by the user.

Potential sources of bias include:

- timing only high-performing or low-performing operators,
- timing only one side/position of a multi-position process,
- timing unusual product conditions and labeling them Normal,
- changing the work method during data collection,
- inconsistent cycle boundaries,
- observer reaction time,
- operator behavior changing because the study is being observed,
- insufficient sampling across normal process variation.

The application's sample-count heuristics are readiness aids, not proof that the sample is statistically representative.

---

# 5. Sample-Count Heuristics Are Not Universal Standards

Current application warnings use approximately:

- 15 Normal selected observations for paced roles,
- 8 Normal selected observations for recurring tasks.

These thresholds help prevent obviously immature baselines. They do **not** establish the required sample count for every process.

A highly variable, high-risk, or infrequently occurring process may require substantially more observation.

---

# 6. Mean, P90, and Variation Limits

The study reports multiple descriptive statistics, but each has limitations.

### Mean

Sensitive to real long/short tails and invalid capture errors if those errors are not reviewed.

### Median

Robust to extreme values but does not directly represent total labor when the distribution is asymmetric.

### P90

The current implementation uses nearest-rank P90. At small `n`, the reported percentile changes in steps and may be dominated by one observation.

### Coefficient of Variation

Useful for relative variation comparison, but no universal CV threshold determines whether a process is stable or acceptable.

---

# 7. Outlier Detection Is Advisory

The outlier system identifies unusual values using median ratios and a log-MAD robust score.

It does not know whether an unusual observation is:

- a capture error,
- a genuine process exception,
- an important tail of normal performance,
- a different method being performed.

Outliers are never automatically excluded. Human review is required.

Excluding valid high-side observations merely to reduce the mean can materially understate labor and capacity risk.

---

# 8. Primary Reference Rate Requirement

A valid Primary Observed Output Rate is required before the staffing model should be used.

Belt-derived rate, placement-opportunity rate, measured bottleneck, and Fastest Available are diagnostic/comparison signals.

They do not automatically redefine the primary staffing basis.

---

# 9. Process-Relationship Dependency

Labor and capacity depend on correct process-event normalization.

An accurate measured cycle time combined with an incorrect `events/reference unit` relationship produces an incorrect labor model.

Likewise, material quantity per process event is separate from timing frequency. Confusing the two can multiply or divide demand incorrectly.

---

# 10. Parallel Position Assumptions

The process-capacity model can divide normalized work across configured parallel positions.

This assumes the positions meaningfully share equivalent process demand.

The model does not automatically account for:

- unequal work distribution between positions,
- one-sided product flow,
- physical interference,
- different operator access,
- skill differences,
- asymmetric equipment constraints.

Review position-level timing before relying on a simple parallel-position interpretation.

---

# 11. Labor Content Is Not the Same as Feasibility

A role can have enough **average labor capacity** and still be operationally infeasible.

Examples:

- a recurring task occurs during the busiest part of the core cycle,
- two requests occur simultaneously,
- a worker must travel farther than the averaged task implies,
- a task requires a skill possessed by only one person,
- buffer capacity is insufficient to absorb the interruption.

This is particularly important for recurring tasks marked as interrupting paced work.

---

# 12. Shared Support Pool Limits

Support load is based on average modeled task seconds/hour versus pooled available labor seconds/hour.

It does not fully model:

- stochastic request arrival,
- simultaneous calls,
- route optimization,
- travel congestion,
- task priority,
- skill constraints,
- break coverage,
- response-time requirements.

A support pool below 100% modeled design load can still fail service-level requirements.

---

# 13. Staffing Outputs Are Planning Estimates

Theoretical Minimum, Design Minimum, and Constraint-Aware Design are different mathematical views.

### Theoretical Minimum

Assumes 100% utilization and perfect rebalancing.

### Design Minimum

Adds the selected design-efficiency allowance but still pools labor before aggregate rounding.

### Constraint-Aware Design

Preserves dedicated-role and support-pool rounding structure, but still cannot model every real work-assignment constraint.

None of these values automatically establishes an approved staffing level.

Review:

- safety,
- ergonomics,
- quality ownership,
- skill requirements,
- required coverage,
- breaks/relief,
- machine interaction,
- site policy,
- actual production trials.

---

# 14. Design Efficiency Is a User Assumption

Design efficiency is not measured automatically from the time study.

Changing it can materially change the design staffing result without changing any observed cycle time.

The selected value should be governed by the intended planning method and local engineering practice.

---

# 15. Bottleneck Model Limits

The measured bottleneck is the minimum modeled capacity across configured/measured paced roles.

It may omit constraints that are not represented as paced roles, such as:

- equipment mechanical limits,
- upstream/downstream starvation/blockage behavior,
- quality holds,
- batch/accumulation logic,
- changeover effects,
- maintenance effects,
- operator interactions not captured in the role model.

Support pools are not treated as paced bottleneck roles; their workload is analyzed separately.

---

# 16. Work-Window / Opportunity Limits

A measured placement opportunity or conveyor work zone is a useful physical comparison, but it does not guarantee that the entire theoretical window is usable manual work time.

Actual usable time may be reduced by:

- product arrival variation,
- reach limits,
- obstruction,
- handoff constraints,
- safe access,
- visual/quality checks,
- task sequencing.

P90 work versus opportunity should therefore be treated as a diagnostic reserve measure.

---

# 17. Yamazumi Limits

The Yamazumi exposes measured work composition and imbalance. It does not automatically solve the line balance.

Moving an element from one role to another may be mathematically attractive but physically impossible or undesirable.

Any rebalance requires separate validation of:

- access,
- precedence,
- motion path,
- safety,
- ergonomics,
- machine timing,
- quality responsibility,
- skill requirements.

---

# 18. Scenario Model Limits

The Quick Scenario model scales aggregate paced work using the current average paced seconds/event.

It assumes added or removed work is reasonably similar to the measured average.

It is not appropriate for a future state where:

- a specific new component has very different work content,
- process routing changes,
- automation changes the method,
- support workload changes nonlinearly,
- different equipment is introduced.

Use a detailed future-state model or the downstream Material Flow & Takt Planner where appropriate.

---

# 19. Rework Labor Boundary in v1.45.1

The current report/model structure retains fields for rework labor compatibility, but the v1.45.1 aggregate `reworkLabor()` calculation currently evaluates to zero.

Therefore, observed rework event duration should **not** be assumed to automatically add rework labor seconds/reference unit to the current staffing result.

If rework labor must be modeled for a decision, represent it using an appropriate explicit measured work definition or other approved method rather than assuming the aggregate rework field is populated.

---

# 20. Data Persistence Limits

The application uses browser local storage for autosave/recovery and the Study Repository for official save/open.

Browser-local recovery can become unavailable if:

- browser data is cleared,
- a different browser/profile/device is used,
- site/origin context changes,
- security policy removes stored permissions/data.

Local autosave is not a substitute for a saved repository package.

---

# 21. Browser / Repository Limits

The current Study Repository connection uses the browser File System Access API and requires supported desktop folder access. The application explicitly directs users to desktop **Microsoft Edge or Google Chrome** for repository access.

Browser or enterprise security policy may require folder permission to be re-granted.

The app verifies local writes, but external OneDrive/SharePoint synchronization is outside the application's control.

---

# 22. Downstream Data Limits

The Analysis Report JSON is flattened for analytics and downstream tools. It is not a substitute for the full-fidelity Study JSON.

Downstream analysis inherits upstream assumptions and errors. A Power BI dashboard or Material Flow & Takt Planner cannot correct a misdefined role relationship or invalid source study automatically.

---

# 23. Engineering Use Disclaimer

Abel Engineering tools are provided as **engineering analysis and planning aids**. Results depend on user-entered data, observations, assumptions, and configuration and should be independently reviewed before being used for production, safety, staffing, financial, regulatory, or equipment-design decisions.

These tools do not replace professional engineering judgment, applicable standards, manufacturer requirements, site procedures, or required safety reviews.

Users are responsible for verifying calculations, inputs, outputs, and suitability for their intended application.

These applications are not intended to function as machine safety systems, safety PLC logic, protective devices, or real-time process-control systems.

---

## Related Documents

- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **REF-AE-CTL-002** — Yamazumi & Standard-Work Analysis
- **REF-AE-CTL-003** — Timing Outlier Review
- **DATA-AE-CTL-001** — Data Handling & Study Repository
