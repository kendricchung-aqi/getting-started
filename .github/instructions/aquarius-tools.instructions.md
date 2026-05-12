---
applyTo: "**", "tools/*"
---

**If applicable, use any `README` or documentation files included with the tools**

# Required tools for AQUARIUS onboarding include (in `tools` with their own folders):
- ProvisioningTool.exe (and related documentation)
- TimeSeriesExporter.exe (EXIM Exporter)
- TimeSeriesImporter.exe (EXIM Importer)
- LocationDeleter
- AQUARIUS.Connect.Provisioning.Utility

*Ensure all of these tools are available and accessible before starting the onboarding process.*

## Credentials
- Credentials for AQTS can be found in `data/aqts_connection.txt`
- Credentials for AQConnect can be found in `data/aqconnect_connection.txt`

*Ensure these credentials are available before starting the onboarding process.*

# Aquarius Tools & Configuration Reference

This document covers the tools used during AQUARIUS onboarding: the ProvisioningTool, EXIM, AQUARIUS Connect, and common scripting utilities.

---

## ProvisioningTool

The **ProvisioningTool** is a console utility for bulk provisioning of AQUARIUS Time-Series (AQTS) configuration using CSV or Excel files. It must be run before any data import.

### Basic Usage

```cmd
ProvisioningTool.exe /Server=<server> /Task="<operation> <item> <file.csv>"
```

Credentials default to `admin`/`admin`. Override with `/Username=` and `/Password=`.

Use the `@options.txt` syntax to avoid complex command lines:

```cmd
ProvisioningTool.exe @mytasks.txt
```

Where `mytasks.txt` contains:

```
# Server credentials
-Server=myserver
-Username=admin
-Password=admin

# Tasks in order
-Task=Create Location locations.csv
-Task=Create TimeSeries timeseries.csv
```

### Supported Operations

`Create`, `Update`, `Delete`, `Export`

**Always export before modifying** — use `Export` to capture the current state as a baseline.

### Key Provisionable Items

| Item | Notes |
|------|-------|
| `Location` | Identifier, name, type, folder, UTC offset, extended attributes |
| `TimeSeries` | Parameter, label, unit, interpolation type, UTC offset, gap tolerance |
| `Parameter` | ID, display name, unit group |
| `Unit` / `UnitGroup` | Required before creating time series that use custom units |
| `Grade` / `Qualifier` / `ApprovalLevel` | Must exist before importing annotated data |
| `LocationFolder` | Folder hierarchy for organizing locations |
| `DerivedSeries` | Child series derived from parent raw series |
| `ExtendedAttribute` | Custom metadata fields on locations or time series |

### Time Series CSV — Critical Columns

| Column | Example | Notes |
|--------|---------|-------|
| `LocationIdentifier` | `05JJ009` | Text ID of the owning location |
| `ParameterId` | `HG` | Use the parameter **ID**, not display name |
| `Label` | `Telemetry` | Must be unique per location/parameter combination |
| `TimeSeriesType` | `Basic` | `Basic` for raw data; `Reflected` for external inputs |
| `UnitId` | `m` | Must exist in AQTS |
| `InterpolationType` | `InstantaneousValues` | Must match the nature of the data |
| `UtcOffset` | `-07:00` | **Cannot be changed after data is imported** |
| `GapToleranceInMinutes` | `60` | Initial gap tolerance |

### Important Rules

- **Parameter IDs are not display names** — use `HG` not `Stage`; use `QR` not `Discharge`
- **UTC offset is permanent** — set it correctly before any import; it cannot be updated later
- **Parent series must be created before child/derived series**
- Run tasks in dependency order: Parameters → Units → Locations → Time Series → Derived Series

---

## EXIM Tools (Also called TimeSeriesExporter and TimeSeriesImporter)

EXIM tools import and export raw time-series data into AQUARIUS Time-Series.

### Required File Structure

You can use the Exporter to extract the required `JSON` files from AQTS as templates for your transformed data.

EXIM expects a **`RawPoints.zip`** archive per time series, organized into a specific folder hierarchy:

```
<LocationIdentifier>/
    TimeSeries/
        <Basic or Reflected (if any)>/
            <ParameterId>.<Label>@<LocationIdentifier>/
                TimeSeriesInfo.json
                RawPoints.zip
    LocationDatum.json
    LocationInfo.json
```

Example for Stage Telemetry at location `05JJ009`:

```
05JJ009/
    TimeSeries/
        Basic/
            HG.Telemetry@05JJ009/
                TimeSeriesInfo.json
                RawPoints.zip
    LocationDatum.json
    LocationInfo.json
```

**Note: When using the Import after Exporting the data folders, make sure to delete the Time-series from all Locations using the LocationDeleter. This will prevent `IdenticalParameterAndLabelException` when using the Importer.**

### RawPoints CSV Format (inside the ZIP)

The CSV inside `RawPoints.zip` must contain at minimum:

| Column | Format                                 | Example                     |
|--------|----------------------------------------|-----------------------------|
| `TIME` | ISO 8601 (with timezone if applicable) | `2023-06-15T14:30:00-05:00` |
| `VALUE` | Numeric                                | `1.452`                     |

Optional columns: `GRADE`, `QUALIFIER`, `APPROVAL`

### Import Rules

- **One time series per ZIP** — do not combine series
- **No duplicate timestamps** — the import will be rejected
- **Timestamps must be strictly ascending** — no out-of-order records
- **No filename artifacts** — avoid suffixes like `_NONE` or extra metadata files in the archive
- **Import parent series before child series** — derived series depend on their parents

### Monitoring EXIM Logs

After each import, review EXIM logs for:

- `ParentNotFound` — a child series was imported before its parent
- `DuplicateTimestamp` — cleansing missed a duplicate
- `UtcOffsetMismatch` — data offset does not match the configured time series offset
- `InvalidFileStructure` — ZIP hierarchy does not match expected format

Always resolve log errors before proceeding with the next location or phase.

---

## AQUARIUS Connect

**AQUARIUS Connect** is the middleware that configures and manages ongoing inbound data feeds into AQTS — the production replacement for manual data imports once the system is live.

### Role in Onboarding

- Connect handles **real-time and scheduled data ingestion** going forward
- The historical migration (via EXIM) must be complete and validated before Connect goes live
- The **delta migration** fills the gap between the end of historical data and the start of Connect ingestion

### Key Configuration Steps

1. Identify all data sources and their connection types (FTP, HTTP, direct logger connection, etc.)
2. Map each data source feed to its target AQUARIUS time series
3. Configure scheduling and polling intervals
4. Test ingestion with a small live data window before full activation
5. Confirm Connect-ingested data aligns with the end of the EXIM-imported historical data (no gap, no overlap)

### Delta Migration

The delta is the time window between:
- **End of historical EXIM import** (e.g., data through Dec 31 of the prior year)
- **Start of Connect ingestion** (e.g., from the current date)

This gap must be filled using the same EXIM import process (extract → transform → import → validate) before Connect is activated, or the time series will have a visible gap in AQUARIUS.

---

## Scripting & Automation

Python and PowerShell are the primary tools for data transformation and workflow automation.

### Python — Common Tasks

```python
import pandas as pd
from datetime import timezone

# Load legacy CSV
df = pd.read_csv('legacy_data.csv', parse_dates=['Timestamp'])

# Normalize timestamps to ISO 8601 with UTC offset
df['TIME'] = df['Timestamp'].dt.tz_localize('America/Edmonton').dt.strftime('%Y-%m-%dT%H:%M:%S%z')

# Remove duplicates (keep first occurrence)
df = df.drop_duplicates(subset='TIME', keep='first')

# Enforce monotonicity
df = df.sort_values('TIME').reset_index(drop=True)

# Rename and select output columns
df = df.rename(columns={'Reading': 'VALUE'})[['TIME', 'VALUE']]

# Write to CSV
df.to_csv('RawPoints.csv', index=False)
```

### PowerShell — ZIP Creation for EXIM

```powershell
# Create the required folder hierarchy and ZIP for EXIM
$location = "05JJ009"
$series   = "HG.Telemetry@05JJ009"
$csvPath  = ".\RawPoints.csv"

$folder = Join-Path $location $series
New-Item -ItemType Directory -Path $folder -Force | Out-Null
Copy-Item $csvPath -Destination $folder

Compress-Archive -Path $folder -DestinationPath ".\$location\$series\RawPoints.zip" -Force
```

### Validation Script Pattern

```python
import requests

# Compare row count in CSV vs points returned by AQUARIUS API
csv_count = len(pd.read_csv('RawPoints.csv'))

response = requests.get(
    f"https://{server}/AQUARIUS/Publish/v2/GetTimeSeriesRawData",
    params={"TimeSeriesUniqueId": unique_id},
    auth=(username, password)
)
aqts_count = len(response.json()['Points'])

print(f"CSV rows: {csv_count} | AQTS points: {aqts_count} | Match: {csv_count == aqts_count}")
```

---

## Common Issues & Resolutions

| Issue | Symptom | Resolution |
|-------|---------|------------|
| Wrong UTC offset | Timestamps shifted in AQUARIUS | Cannot fix after import — must delete series and re-create |
| Duplicate timestamps | EXIM import rejected | Deduplicate in transformation step; re-build ZIP |
| Non-monotonic timestamps | EXIM import rejected | Sort by timestamp in transformation step |
| Parent not found | Child import fails | Import parent series first; check parent label/location ID |
| Wrong parameter ID | Series not found in EXIM | Use parameter ID (`HG`), not display name (`Stage`) |
| `_NONE` suffix in filenames | EXIM can't parse file | Strip all filename artifacts in transformation step |
| Gap between historic and Connect data | Visible hole in AQUARIUS | Perform delta migration before activating Connect |
