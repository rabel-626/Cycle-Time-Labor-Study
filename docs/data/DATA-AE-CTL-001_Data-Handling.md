---
documentId: DATA-AE-CTL-001
title: Cycle Time & Labor Study — Data Handling & Study Repository
tool: Cycle-Time-Labor-Study
documentVersion: 1.1
toolVersion: 1.45.1
status: Draft for Review
updated: 2026-09-08
---

# Data Handling & Study Repository

## 1. Purpose

This document describes how the **Abel Engineering Cycle Time & Labor Study v1.45.1** stores, recovers, saves, opens, and exports study information.

It distinguishes three concepts that should not be confused:

1. **active browser state** — the study currently open in the application,
2. **browser autosave/recovery** — local crash-recovery information,
3. **Study Repository package** — the official saved engineering record used by the current workflow.

---

# 2. Processing Model

The Cycle Time & Labor Study is a browser application. Study calculations are performed in the user's browser.

The application does not require a remote calculation server to compute cycle-time/labor results.

Official file persistence in v1.45.1 uses a user-selected local **Study Repository** folder through supported desktop browser folder access.

If that folder is synchronized by OneDrive, SharePoint/Teams synchronization is handled by the **OneDrive client**, not by the Cycle Time application itself.

---

# 3. Official Study Repository

The application accepts a repository root named exactly:

- `Engineering Study Hub - Study Repository`, or
- `Study Repository`.

The repository schema defines standard routes including:

```text
01_CYCLE_TIME
02_DOWNTIME
03_TAKT_MATERIAL_FLOW
04_MULTI_STUDY_DOWNTIME
90_TEMPLATES
```

The Cycle Time application writes into `01_CYCLE_TIME`.

---

# 4. Cycle Time Folder Structure

The current route is:

```text
Study Repository/
└── 01_CYCLE_TIME/
    └── PLANT/
        └── LINE/
            └── YYYY/
                └── YYYY-MM/
                    └── CT-RECORD-ID/
```

The Plant and Line values entered in the study contribute to the repository path. Maintain consistent naming if records are expected to group together.

---

# 5. Browser Requirement for Repository Access

The current application uses the browser **File System Access API** for direct folder read/write access.

If that interface is unavailable, the application instructs the user that Study Repository access requires desktop:

- Microsoft Edge, or
- Google Chrome.

Enterprise browser policies may affect whether access is permitted or remembered.

The user must grant read/write permission to the selected repository.

---

# 6. Repository Connection Validation

When connecting, the application validates that:

- a repository folder is selected,
- its root name is approved,
- read/write permission is granted,
- the `_SYSTEM/LIBRARY_SCHEMA.json` repository schema is compatible,
- required route folders can be created when initializing,
- the root is writable.

A temporary write probe is used to verify local write access.

---

# 7. Saved Study Package

Selecting **SAVE STUDY** writes a record package into the calculated Cycle Time folder.

The package uses a Cycle Time record ID and creates the following files.

## 7.1 Authoritative Study JSON

```text
CT-RECORD-ID__STUDY.json
```

Purpose:

- full-fidelity save/resume,
- audit/history source,
- contains study setup and raw observation structures.

Current schema identity:

```text
Schema: Abel-Engineering.CartonerStudy
Schema version: 16
Application version: 1.45.1
```

This is the authoritative continuation record.

## 7.2 Downstream Analysis Report JSON

```text
CT-RECORD-ID__REPORT.json
```

Purpose:

- flattened analytical package,
- Power BI/downstream analysis,
- companion Material Flow & Takt workflow.

Current report schema identity:

```text
Schema: Abel-Engineering.CartonerPowerBIReport
Schema version: 16
```

The report contains flattened arrays and calculated summaries. It is not the authoritative resume file.

## 7.3 Timing Samples Excel Workbook

```text
CT-RECORD-ID__TIMING-SAMPLES.xlsx
```

Created when timing samples exist.

Purpose:

- spreadsheet review,
- tabular timing-sample analysis,
- convenient external inspection.

The workbook is a derivative export; the Study JSON remains the authoritative full-fidelity record.

## 7.4 Package Manifest

```text
CT-RECORD-ID__MANIFEST.json
```

Current manifest type:

```text
AbelEngineering.StudyPackageManifest
Schema version: 1
Record type: CYCLE_TIME
```

The manifest records study/record identity, repository relative path, save time, and the output inventory.

---

# 8. Local Write Verification

The repository save process writes and then re-reads JSON outputs to verify that the stored JSON matches the generated package.

The Study and Report output entries include SHA-256 hashes in the save-package inventory.

The application reports the package as **Saved & verified** after successful local writes.

> **Important:** This verifies local file creation. It does not independently verify that an external cloud synchronization service has completed upload to SharePoint/Teams.

---

# 9. Saving During Active Capture

If timing capture is active when **SAVE STUDY** is requested, the application warns the user and offers to save a **frozen snapshot without fabricating the unfinished timer/sequence record**.

This prevents an in-progress timer from being converted into a false completed observation merely because a save occurred.

For a clean study record, prefer saving between observations when practical.

---

# 10. Opening a Saved Study

**OPEN STUDY** scans the Cycle Time repository route for files ending in:

```text
__STUDY.json
```

The application validates that the selected record uses the Cycle Time Study schema before loading it.

When a different Study ID is opened while the active browser contains study data, the application prompts before replacing the current local autosave.

If the selected file appears older than the currently open copy of the same Study ID, an additional confirmation is requested.

---

# 11. Legacy File Load

**LEGACY FILE LOAD** exists only for Cycle Time Study JSON created outside the current repository workflow.

The original legacy file/folder is **not** adopted as a new save destination.

After loading a legacy study:

```text
SAVE STUDY → writes the current study into 01_CYCLE_TIME
```

This keeps one official save destination and avoids ambiguous storage behavior.

---

# 12. Browser Autosave and Recovery

The application maintains local browser state for crash/interruption recovery.

Current local-storage keys include:

```text
Abel-Engineering_CARTONER_STUDY_V1
Abel-Engineering_CARTONER_STUDY_V1_BACKUP
Abel-Engineering_CARTONER_RUNTIME_RECOVERY_V1
Abel-Engineering_CARTONER_DEVICE_ID
```

The application maintains:

- a primary autosave,
- a rolling backup when practical,
- runtime-recovery information for active timing.

If autosave fails, the application is designed to lock further capture rather than silently continue collecting data that cannot be persisted safely.

> **Important:** Local autosave is a recovery mechanism, not the official completed-study record.

---

# 13. Remembered Repository Folder Handle

The current application can remember the selected repository folder handle in browser-managed storage so the same root can be reused when browser permission remains available.

The current implementation uses an IndexedDB database named:

```text
AbelEngineeringStudyRepositoryHandles
```

with a saved repository-root handle.

The browser may still require the user to grant permission again after restart, security-policy change, or browser-storage cleanup.

---

# 14. Data That Can Make Autosave Unavailable

Browser-local recovery may become inaccessible after:

- clearing browser/site data,
- using a different browser,
- changing browser profiles,
- changing devices,
- opening the application under a different origin/context,
- enterprise cleanup/security policies,
- manual reset of browser storage.

Always save important studies to the Study Repository.

---

# 15. Starting a New Study While Keeping Setup

**START NEW STUDY — KEEP LINE SETUP** is intended for a new observed run on the same general line configuration.

It creates a new Study ID and preserves configured line structure, including:

- roles,
- position counts,
- Areas,
- support-task definitions,
- machine/pitch setup,
- design efficiency,
- plant/line/equipment,
- observer defaults.

It clears run/capture-specific information including:

- operator records,
- timing samples,
- belt-speed trials,
- placement-window trials,
- machine-rate samples,
- job/run identity,
- total headcount,
- notes,
- report sequence.

The next report sequence returns to `R001`.

> **Required practice:** Save the current study before creating the next Study ID if the current record must be retained.

---

# 16. Full Reset

**FULL RESET — CLEAR EVERYTHING** creates a completely blank active study.

It clears the active browser study's:

- mapping standards,
- roles,
- positions,
- support tasks,
- machine setup,
- operators,
- captured data.

The application uses multiple confirmations because this replaces the active local autosave.

A previously saved repository package is not deleted merely because the active browser study is reset.

---

# 17. Process Template and Mapping Files

The Cycle Time tool also supports reusable configuration data separate from a full study.

## Process Template

```text
Schema: Abel-Engineering.CycleTimeLaborRoleStations
Schema version: 5
```

Purpose:

- reuse configured Areas/roles/stations,
- recurring/support task definitions,
- standard-work structures,
- without carrying timing observations as the active study baseline.

## Dropdown Mapping

```text
Schema: Abel-Engineering.CartonerMapping
Schema version: 2
```

Purpose:

- reuse controlled dropdown vocabulary/standards,
- without replacing the current Study ID or timing observations.

These configuration files should not be confused with a saved Study package.

---

# 18. Study vs Report vs Template

| File type | Full raw study? | Used to resume? | Contains calculated report views? | Intended use |
| --- | :---: | :---: | :---: | --- |
| **Study JSON** | Yes | **Yes** | Some state/calculated context | Authoritative study record |
| **Report JSON** | Flattened/derived | No | **Yes** | Analytics/downstream tools |
| **Timing XLSX** | Tabular derivative | No | Selected tables | Spreadsheet review |
| **Process Template** | Definitions only | No | No | Reuse line/process structure |
| **Mapping JSON** | Vocabulary only | No | No | Reuse dropdown standards |
| **Manifest** | Inventory/metadata | No | No | Package traceability |

---

# 19. Data Minimization and Operator Privacy

Use non-sensitive operator aliases such as:

```text
OP-01
OP-02
OP-03
```

when individual identity is not required.

Avoid entering unnecessary personal information into:

- operator alias fields,
- study notes,
- free-text task descriptions,
- reviewer notes.

The tool is designed to support operator variation analysis without requiring real names.

Follow organizational policy for any production, personnel, customer, SKU, or other sensitive information stored in study records.

---

# 20. OneDrive / SharePoint Synchronization

When the approved Study Repository resides in a OneDrive-synchronized location:

```text
Cycle Time application
        ↓
verified local repository files
        ↓
OneDrive client
        ↓
SharePoint / Teams synchronization
```

The application does not control:

- OneDrive sign-in,
- network availability,
- cloud upload timing,
- sync conflict handling,
- SharePoint retention/version policy.

Confirm synchronization status through the organization's normal OneDrive/SharePoint controls when cloud availability matters.

---

# 21. Recommended Data-Safety Practice

1. Connect the correct Study Repository before field collection.
2. Confirm Plant and Line values before the first official save.
3. Save after meaningful setup milestones.
4. Save at the end of each major observation session.
5. Do not clear browser data before the repository save is verified.
6. Use Study JSON as the authoritative resume record.
7. Preserve the package manifest with the study outputs.
8. Do not manually rename individual files inside a package unless the data-management process is designed to preserve package relationships.
9. Confirm OneDrive/cloud synchronization separately when required.
10. Use **START NEW STUDY — KEEP LINE SETUP** only after the prior study is safely saved.

---

# 22. Data Handling Boundaries

The tool does not provide a guarantee of:

- cloud backup,
- SharePoint retention,
- enterprise access control,
- confidentiality classification,
- regulatory record retention,
- recovery after manual deletion of repository files.

Those controls belong to the host computer, browser, filesystem, OneDrive/SharePoint environment, and organizational data-governance process.

---

## Related Documents

- **WI-AE-CTL-001** — Study Procedure
- **REF-AE-CTL-001** — Cycle Time & Labor Methodology
- **LIM-AE-CTL-001** — Limitations & Engineering Boundaries
