---
applyTo: "**"
---

# Aquarius Data Migration Reference

This document covers the discovery, field mapping, data extraction, transformation, and validation specifics for migrating historical data into AQUARIUS Time-Series.

---

## Discovery & Business Analysis

### Legacy System Inventory

Before planning a migration, gather the following from the client or BA document:

| Item | Questions to Answer |
|------|-------------------|
| **Legacy system** | What system holds the data? (Hydstra, OTT, SQL DB, flat files?) |
| **Locations** | How many stations/sites? What are their identifiers in the legacy system? |
| **Parameters** | What parameters are measured at each location? What are the legacy parameter codes? |
| **Date ranges** | What is the earliest date to migrate? Are there gaps in coverage? |
| **Data frequency** | What is the recording interval? (5-min, 15-min, hourly, daily?) |
| **Time zones** | What time zone is the data stored in? Is DST observed? |
| **Volume** | Estimated total number of records per series? |
| **Metadata** | Are grades, qualifiers, or approval levels recorded in the legacy system? |
| **Naming** | What naming conventions should be used in AQUARIUS? |

### Field Mapping Document

Produce a field mapping table before any technical work begins. Example:

| Legacy Field | Legacy Value | AQUARIUS Field | AQUARIUS Value |
|-------------|-------------|----------------|----------------|
| Station ID | `WQ001` | LocationIdentifier | `WQ-001` |
| Parameter | `STAGE` | ParameterId | `HG` |
| Unit | `metres` | UnitId | `m` |
| Interval | 15 min | GapToleranceInMinutes | `20` |
| Time Zone | `MST (UTC-7)` | UtcOffset | `-07:00` |
| Data grade | `A` | Grade | `A` |

Share and confirm this mapping with the client before proceeding.

---

## Common Legacy Systems

### Hydstra

Hydstra stores hydrological data in a proprietary database. Common export approach:

1. Use Hydstra's built-in export utilities or the `HYCSV` command to produce CSV exports
2. Typical Hydstra CSV format:

```
Date,Time,Value,Quality
15/06/2023,14:30,1.452,10
15/06/2023,14:45,1.461,10
```

3. Watch for:
   - Date format `DD/MM/YYYY` — convert to ISO 8601
   - Quality codes (e.g., `10` = good, `30` = estimated) — map to AQUARIUS grade/qualifier
   - Australian or local time zones — confirm DST handling with client
   - Multiple parameters in a single export file — split into per-series files

### OTT Data Loggers

OTT logger software exports typically produce:

- Tab-delimited or CSV files per logger/channel
- Logger-local timestamps (may not align with AQUARIUS time series UTC offset)
- Proprietary quality flags

Steps:
1. Export from OTT Hydras or similar logger management software
2. Confirm the logger clock's time zone and any known drift
3. Map logger channels to AQUARIUS parameter IDs

### Custom Databases (SQL)

For clients with bespoke data stores:

1. Obtain read access or a data extract from the client
2. Write SQL queries to produce one result set per time series:

```sql
SELECT
    FORMAT(timestamp_col, 'yyyy-MM-ddTHH:mm:ss') AS TIME,
    value_col AS VALUE
FROM measurements
WHERE station_id = 'WQ001'
  AND parameter = 'STAGE'
  AND timestamp_col BETWEEN '2010-01-01' AND '2023-12-31'
ORDER BY timestamp_col ASC;
```

3. Export to CSV and proceed with standard transformation

---

## Data Transformation Checklist

Before building EXIM archives, verify each series passes all of the following:

### Timestamp Checks

- [ ] All timestamps are in ISO 8601 format (`YYYY-MM-DDTHH:MM:SS`)
- [ ] UTC offset is applied correctly (fixed offset per series — no DST shifts mid-series unless the series UTC offset accommodates it)
- [ ] No timestamps outside the agreed date range
- [ ] No `NULL` or blank timestamps

### Data Integrity Checks

- [ ] No duplicate timestamps (exact duplicates or near-duplicates within the resolution)
- [ ] Timestamps are in strictly ascending order (monotonic)
- [ ] No `NULL` or blank values (replace with gaps if data is genuinely absent)
- [ ] Values are within a plausible physical range for the parameter
- [ ] No string values in the `VALUE` column (e.g., `"---"`, `"N/A"`)

### EXIM Structure Checks

- [ ] One CSV file per time series
- [ ] Column headers are exactly `TIME` and `VALUE` (and optionally `GRADE`, `QUALIFIER`)
- [ ] Each CSV is packaged into a `RawPoints.zip`
- [ ] ZIP is placed in the correct folder hierarchy:
  ```
  <LocationIdentifier>/<ParameterId>.<Label>@<LocationIdentifier>/RawPoints.zip
  ```
- [ ] No extra files in the ZIP archive
- [ ] No filename artifacts (`_NONE`, extra extensions, etc.)

---

## Validation Reference

### What to Validate

After EXIM import, validate each time series against its source data:

| Check | Method |
|-------|--------|
| **Point count** | Row count in CSV vs. point count from AQUARIUS API |
| **Date range** | First/last timestamp in CSV vs. AQUARIUS |
| **Min/Max values** | Statistical summary comparison |
| **Aggregated totals** | Daily or monthly sums compared between legacy and AQUARIUS |
| **Visual review** | Plot the series in AQUARIUS and look for spikes, gaps, flat lines |

### AQUARIUS Publish API — Useful Endpoints

```
GET /AQUARIUS/Publish/v2/GetTimeSeriesRawData?TimeSeriesUniqueId=<id>
GET /AQUARIUS/Publish/v2/GetTimeSeriesDescriptionList?LocationIdentifier=<id>
GET /AQUARIUS/Publish/v2/GetLocationData?LocationIdentifier=<id>
```

### Validation Report Template

Produce this summary per time series (or consolidated per location):

```
Location:       05JJ009
Parameter:      HG (Stage)
Label:          Telemetry
Date Range:     2010-01-01 to 2023-12-31

Source Records: 525,601
AQTS Points:    525,601     ✓ MATCH

Source Start:   2010-01-01T00:00:00-07:00
AQTS Start:     2010-01-01T00:00:00-07:00  ✓ MATCH

Source End:     2023-12-31T23:45:00-07:00
AQTS End:       2023-12-31T23:45:00-07:00  ✓ MATCH

Source Max:     3.214 m
AQTS Max:       3.214 m     ✓ MATCH

Known Issues:   Data gap 2018-07-03 to 2018-07-10 (logger offline — confirmed with client)
```

### Handling Validation Failures

| Failure | Likely Cause | Action |
|---------|-------------|--------|
| Point count mismatch | Duplicates removed during cleansing | Document records removed; confirm with client |
| Date range mismatch | Partial import or export cutoff | Check EXIM logs; re-import if truncated |
| Value mismatch | Unit conversion error or wrong series targeted | Re-check field mapping; re-import |
| Gap in AQUARIUS | Data truly absent or transformation dropped records | Re-audit source; document if confirmed gap |

---

## Delta Migration

The delta is the data between the end of the historical EXIM import and the start of AQUARIUS Connect ingestion.

### How to Identify the Delta

1. Note the **last timestamp** successfully imported via EXIM (e.g., `2023-12-31T23:45:00`)
2. Note the **first timestamp** captured by AQUARIUS Connect after activation (e.g., `2024-01-15T08:00:00`)
3. The delta window is everything in between

### Delta Migration Process

1. Extract the delta period from the legacy system using the same extraction method as the historical migration
2. Apply the same transformation and cleansing steps
3. Build `RawPoints.zip` archives for the delta period only
4. Import via EXIM — AQUARIUS will append to existing data without overwriting
5. Validate: confirm no gap or overlap between historical data, delta data, and Connect ingestion

### Common Delta Pitfalls

- **Overlap**: Delta import overlaps with data already in AQUARIUS — check for duplicate timestamps at boundaries
- **Gap**: Delta does not fully cover the window — re-extract and re-import the missing range
- **Connect already active**: If Connect is already ingesting, pause it during delta import to avoid collisions
