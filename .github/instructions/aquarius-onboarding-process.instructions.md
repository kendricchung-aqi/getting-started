---
applyTo: "**"
---

# Aquarius Onboarding Process

This document defines the 8-phase implementation framework used to onboard clients onto AQUARIUS Time-Series (AQTS) and related AQI products. Follow these phases in order for every customer engagement.

---

## Phase 1 — Discovery & Planning

**Goal**: Understand the client's environment, data, and requirements before any configuration or migration work begins.

### Key Tasks

- Review the Business Analysis (BA) document or conduct discovery sessions with the client
- Identify all **locations** (monitoring stations/sites) to be migrated or configured
- Identify all **parameters** per location (e.g., stage, discharge, water quality constituents)
- Confirm **time zones and UTC offsets** for each location and data source
- Determine **data frequency and volume** (e.g., 15-minute intervals, years of history)
- Confirm scope:
  - Which historical date ranges to migrate
  - Which metadata must be preserved (grades, qualifiers, approvals)
  - Naming conventions and standards to apply in AQUARIUS
- Identify data quality risks:
  - Duplicate timestamps in legacy data
  - Gaps or non-monotonic sequences
  - Ambiguous parameter labels or units
  - Multiple time zones within the same dataset

### Deliverables

- Migration plan document
- Field mapping documentation (legacy field → AQUARIUS equivalent)
- Agreed location identifiers, parameter IDs, and labels for AQUARIUS
- Known risks and exclusions list

---

## Phase 2 — Aquarius Configuration

**Goal**: Ensure AQUARIUS is fully configured to receive data before any import is attempted. Import will fail if the target locations and time series do not exist.

### Key Tasks

Use the **ProvisioningTool** to create or update:

- **Locations**: identifiers, names, types, folders, extended attributes
- **Time Series**: parameter, label, unit, interpolation type, UTC offset, gap tolerance
- **Parameters**: IDs, display names, unit groups
- **Units and Unit Groups**: ensure correct units are available
- **Grades, Qualifiers, Approval Levels**: configure before importing annotated data

### Critical Checks

- Confirm **parent–child relationships** are correct — raw (parent) series must exist before derived/corrected (child) series
- Verify **parameter IDs** match what will be used in EXIM import files (use the ID `HG`, not the display name `Stage`)
- Confirm **UTC offsets** on each time series — this cannot be changed after data is imported
- Confirm **interpolation types** match the nature of the data (e.g., `PrecedingTotals` for rainfall)
- Align time series configuration with EXIM folder hierarchy requirements

### Why This Matters

Even perfectly clean data will fail to import if the target time series does not exist or is misconfigured. Configuration must be complete and validated before moving to data extraction.

---

## Phase 3 — Data Extraction from Legacy Systems

**Goal**: Retrieve all required historical data from the client's source system in a usable format.

### Common Legacy Systems

- **Hydstra**: Export CSVs using Hydstra export tools or custom scripts
- **OTT data loggers**: Export from OTT Hydras or similar logger software
- **Custom databases**: SQL queries or API exports
- **Flat files**: Existing CSVs, Excel files, or proprietary formats

### Key Tasks

- Run legacy export scripts or tools to produce data files
- Ensure exports include correct timestamps and values for the agreed date ranges
- Confirm consistent file formats (column headers, delimiters, date formats)
- Organize extracted files by location and parameter — one file per time series where possible
- Verify date range coverage matches the migration plan

---

## Phase 4 — Data Transformation & Cleansing

**Goal**: Convert legacy data into EXIM-compliant format. This is the most technically demanding phase.

### Timestamp Normalization

- Convert all timestamps to **ISO 8601 format** (e.g., `2023-06-15T14:30:00`)
- Apply correct **UTC offset** as defined in the migration plan
- Handle daylight saving time transitions carefully — AQUARIUS expects fixed offsets per series
- Verify no timestamps fall outside the agreed date range

### Data Cleansing

- **Remove duplicate timestamps** — EXIM will reject files with duplicates
- **Enforce strict time monotonicity** — timestamps must be in strictly ascending order
- **Handle missing or invalid values** — replace with gaps or flag as appropriate
- **Validate value ranges** — flag obvious outliers for client review

### EXIM Format Requirements

- One time series per file
- Correct column headers: `TIME` and `VALUE` (plus optional `GRADE`, `QUALIFIER`, `APPROVAL`)
- Organized into the correct **folder hierarchy** for EXIM:
  ```
  <LocationIdentifier>/<ParameterId>.<Label>@<LocationIdentifier>/RawPoints.zip
  ```
- **One `RawPoints.zip` per time series** — do not combine multiple series in one archive
- No extraneous metadata or filename artifacts (e.g., `_NONE` suffixes)

### Tools Commonly Used

- **Python scripts**: timestamp parsing, deduplication, format conversion, ZIP creation
- **PowerShell**: file organization, batch processing, log parsing
- **CSV utilities**: column reordering, header normalization

---

## Phase 5 — Import Execution (EXIM)

**Goal**: Load cleansed data into AQUARIUS Time-Series using EXIM tools.

### Key Tasks

- Use AQUARIUS EXIM tools to import raw time series data
- **Sequence imports carefully**:
  - Import parent (raw) series before child (derived/corrected) series
  - Import one location at a time for large or complex migrations
- Monitor EXIM logs after each import for:
  - Parent import failures (child imports depend on these succeeding)
  - Time zone conversion issues or offset mismatches
  - Validation errors (duplicates, out-of-range values)
- Re-import after correcting any failures — do not leave partial imports

### Common Import Failures

| Error | Likely Cause | Resolution |
|-------|-------------|------------|
| Parent series not found | Child imported before parent | Reorder imports; import parent first |
| Duplicate timestamp | Legacy data not fully deduplicated | Rerun cleansing step |
| UTC offset mismatch | Time series offset differs from data offset | Correct in transformation; re-import |
| File structure invalid | Wrong folder hierarchy or ZIP format | Rebuild archive per EXIM spec |

---

## Phase 6 — Validation & Quality Assurance

**Goal**: Confirm imported data in AQUARIUS matches the legacy source data.

### Post-Import Checks

- Confirm **start and end timestamps** match legacy export date ranges
- Verify **value counts** per time series
- Check for **duplicate or shifted timestamps** in AQUARIUS
- Compare **totals or aggregates** (e.g., daily means, monthly sums) between legacy and AQUARIUS

### Validation Methods

- Visual plots in AQUARIUS — look for obvious discontinuities, spikes, or flat lines
- Scripted comparisons: legacy CSV row count vs. AQUARIUS API point count
- Statistical summaries: min, max, mean comparison per time series

### Deliverables

- Validation report per time series (or consolidated summary)
- List of any warnings, failures, known gaps, or excluded data
- Client-ready comparison showing legacy vs. AQUARIUS results

### Iteration

This phase often repeats. Fix issues, re-import affected series, re-validate. Do not proceed to client review until validation is accepted internally.

---

## Phase 7 — Client Review & Sign-Off

**Goal**: Confirm with the client that migrated data meets requirements and obtain formal acceptance.

### Key Tasks

- Walk the client through migrated data in AQUARIUS
- Present validation results and comparison to legacy data
- Explain any known limitations, exclusions, or data quality issues found during migration
- Answer technical questions and make any agreed adjustments
- **Obtain formal written sign-off** on the migrated data before closing the migration

---

## Phase 8 — Documentation & Handover

**Goal**: Ensure the client can operate AQUARIUS independently and that the engagement is fully documented.

### Deliverables

- **Migration summary**: what was migrated, date ranges, any exclusions
- **Known issues list**: unresolved data quality issues, limitations, future work items
- **Configuration documentation**: locations, time series, parameters provisioned; Connect configuration
- Updated internal records (CRM, project tracker)
- Support for go-live activities if applicable
- Based off initial discovery (number of locations, time-series, etc to be created), provide a assertion of what was created based off the source data (B.A, Statement of Work, Sample source data, etc).
- It should be clear what the customer can expect based off of their source data and what was created in AQTS. This will help to set expectations for the customer after the work has been completed or following a checkpoint with the customer.

### Transition

- Hand off to the operations or support team with full context
- Confirm the client has access to AQUARIUS support resources
- Schedule a post-go-live check-in if required
