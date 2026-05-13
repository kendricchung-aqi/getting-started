---
applyTo: "**", "tools/*"
---

**If applicable, use any `README` or documentation files included with the tools**
**If applicable, store the data used by the tools in the `data` folder**

# Required tools for AQUARIUS onboarding include (in `tools` with their own folders):
- ProvisioningTool.exe (and related documentation)
- TimeSeriesExporter.exe (EXIM Exporter)
- TimeSeriesImporter.exe (EXIM Importer)
- LocationDeleter
- AQUARIUS.Connect.Provisioning.Utility (`README` and examples are located in the `Examples` folder)

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

### Provisioning Connect with the Provisioning Utility

Use `AQUARIUSConnectProvisioningUtility.exe` (located in `tools/AQUARIUS.Connect.Provisioning.Utility/`) to bulk-configure Connect from a JSON file. Example files and a README are in the `Examples/` subfolder.

#### Basic command

```cmd
AQUARIUSConnectProvisioningUtility.exe ^
  --json=MyConfig.json ^
  --hostname=my.connect.url ^
  --port=80 ^
  --username=myconnectuser ^
  --password=myconnectpassword ^
  --log=ConnectProvisioning.log
```

> **Warning — `--restore` flag:** Adding `--restore` resets the target Connect system to its initial state **before** applying the JSON. This **deletes all locations, connectors, and schedules**. Omit `--restore` to add/update entities without deleting existing ones.

#### JSON structure

The provisioning JSON has these top-level sections:

| Section | Purpose |
|---------|---------|
| `locations` | Connect locations (monitoring sites) |
| `connectors` | Data ingest connectors linked to one or more locations |
| `schedules` | Named schedules that trigger connectors |
| `extractionRuleProfiles` | Reusable extraction driver configurations |
| `inboundConnectionRuleProfiles` | Reusable inbound connection configurations |
| `exportRuleProfiles` | Reusable AQTS export configurations |
| `outboundConnectionRuleProfiles` | Reusable outbound connection configurations |

#### Locations

```json
"locations": [
    {
        "identifier": "TASBRIDGE",
        "name": "Tasman Bridge",
        "utcOffset": "10:00:00"
    }
]
```

The `identifier` should match the location identifier in AQUARIUS Time-Series.

#### Connectors

Each connector specifies:
- `location` — links the connector to a site (or omit and set `location` per data set for multi-site connectors)
- `extractionRule` — how to parse incoming data (driver + rule profile + optional overrides)
- `inboundConnectionRule` — where to pull data from (FTP, file system, database, HTTP, etc.)
- `dataSets` — individual time series being ingested, each with one or more `exportTargets`
- `schedules` — named schedules that trigger the connector (omit for manual-only)

**Single-location connector — FTP + XML (no schedule, manual only):**

```json
{
    "name": "Alpha Connector",
    "location": "ALPHA",
    "extractionRule": {
        "driver": "XML File Extraction Driver",
        "ruleProfile": "Default",
        "settingOverrides": {
            "XsltPath": "\\\\NAS\\Config\\DataSchema.xslt",
            "Source UTC Offset": "00:00:00"
        }
    },
    "inboundConnectionRule": {
        "driver": "FTP Inbound Connection Driver",
        "ruleProfile": "Default",
        "settingOverrides": {
            "FTP server": "ftp.alpha.com",
            "User name": "employee1",
            "Password": "beta",
            "Enable SSL": true,
            "Paths": ["/Data/Alpha1.xml", "/Data/Alpha2.xml"]
        }
    },
    "dataSets": [
        {
            "identifier": "Precip Increm.Primary",
            "name": "Rainfall (Primary)",
            "exportTargets": [
                {
                    "exportRule": {
                        "driver": "AQUARIUS Time-Series Export Driver",
                        "ruleProfile": "Default",
                        "settingOverrides": {
                            "AQUARIUS Time-Series username": "aquser",
                            "AQUARIUS Time-Series password": "password",
                            "Location identifier": "SiteOne"
                        }
                    },
                    "outboundConnectionRule": {
                        "driver": "HTTP Outbound Connection Driver",
                        "ruleProfile": "Default",
                        "settingOverrides": {
                            "Address": "https://aquarius.alpha.com/"
                        }
                    }
                }
            ]
        }
    ]
}
```

**Multi-location connector — database, per-dataset location (runs hourly):**

```json
{
    "name": "Meteorology Database",
    "schedules": ["Hourly"],
    "extractionRule": {
        "driver": "Database Extraction Driver",
        "ruleProfile": "Meteorology Time-Series Data"
    },
    "inboundConnectionRule": {
        "driver": "Database Inbound Connection Driver",
        "ruleProfile": "Meteorology DB Server 1"
    },
    "dataSets": [
        {
            "identifier": "TASBRIDGE_PRECIP",
            "location": "TASBRIDGE",
            "name": "Precipitation",
            "exportTargets": [
                {
                    "exportRule": {
                        "driver": "AQUARIUS Time-Series Export Driver",
                        "ruleProfile": "AQTS Acquisition",
                        "settingOverrides": {
                            "Parameter identifier": "Precip Increm",
                            "Time-series label": "Primary"
                        }
                    },
                    "outboundConnectionRule": {
                        "driver": "HTTP Outbound Connection Driver",
                        "ruleProfile": "AQTS Server 1"
                    }
                }
            ]
        }
    ]
}
```

#### Data set identifiers

For the **AQUARIUS Time-Series Export Driver**, the data set `identifier` is parsed as `<ParameterId>.<Label>` (e.g., `Precip Increm.Primary` → parameter `Precip Increm`, label `Primary`). Use `Parameter identifier` and `Time-series label` in `settingOverrides` when the identifier format does not follow this convention.

If `Location identifier` is omitted from the export target, it defaults to the connector's `location`.

#### Schedules

```json
"schedules": [
    {
        "name": "Real-time",
        "triggers": [{"type": "Daily", "utcOffset": "10:00:00", "stepTimeOfDay": "00:01:00"}]
    },
    {
        "name": "Hourly",
        "triggers": [{"type": "Daily", "utcOffset": "10:00:00", "stepTimeOfDay": "01:00:00"}]
    }
]
```

`stepTimeOfDay` is the repeat interval: `00:01:00` = every 1 minute, `01:00:00` = every hour.

#### Rule profiles

Rule profiles are reusable named driver configurations. They have `driver`, `name`, `settings`, and optionally `ruleCanOverride` (which settings connectors are allowed to override via `settingOverrides`).

**Text file extraction (CSV with regex):**

```json
{
    "driver": "Text File Extraction Driver",
    "name": "CSV Format",
    "settings": {
        "Text parsing expression": "^\\s*(?<Year>\\d{4})-(?<Month>\\d{2})-(?<Day>\\d{2})\\s+(?<Time>\\d{2}:\\d{2}:\\d{2})(\\s*,\\s*(?<Value>[\\d.+-]+)){$SensorP1}"
    }
}
```

`$SensorP1` is replaced at runtime by the data set identifier. Named groups `Year`, `Month`, `Day`, `Time`, `Value` are reserved by the Text File Extraction Driver.

**Database extraction (SQL):**

```json
{
    "driver": "Database Extraction Driver",
    "name": "Meteorology Time-Series Data",
    "settings": {
        "SQL Query": "SELECT \"Timestamp\", \"Value\" FROM \"TimeSeriesValue\" WHERE \"Identifier\" = '$DataSetId' AND \"Timestamp\" >= '$StartPoint(\"yyyy-MM-dd HH:mm:ss\")'",
        "Time stamp column": "Timestamp",
        "Value column": "Value"
    }
}
```

`$DataSetId` is replaced by the data set identifier; `$StartPoint(...)` provides the last successfully ingested timestamp.

**Hot folder inbound connection:**

```json
{
    "driver": "File System Inbound Connection Driver",
    "name": "Hot Folder",
    "settings": {
        "File queue sort Order": "File Last Modified Ascending",
        "More or Delete Source Files After Extraction": "Move Extracted Only",
        "Path when Extraction Succeeds": "Processed"
    },
    "ruleCanOverride": {"Source paths": true}
}
```

**Database inbound connection:**

```json
{
    "driver": "Database Inbound Connection Driver",
    "name": "Meteorology DB Server 1",
    "settings": {
        "Database Provider": "ODBC",
        "Connection String": "Driver={SQL Server};Server=dbserver1.local;Database=Meteorology"
    }
}
```

**AQTS export rule profile:**

```json
{
    "driver": "AQUARIUS Time-Series Export Driver",
    "name": "AQTS Acquisition",
    "settings": {
        "AQUARIUS Time-Series username": "aquser",
        "AQUARIUS Time-Series password": "password"
    },
    "ruleCanOverride": {"Parameter identifier": true, "Time-series label": true}
}
```

**HTTP outbound connection (AQTS server):**

```json
{
    "driver": "HTTP Outbound Connection Driver",
    "name": "AQTS Server 1",
    "settings": {
        "Address": "http://aqserver1.local/"
    }
}
```

`Address` is the base URL of the AQTS server — omit the `/AQUARIUS/` suffix.

#### Available drivers

| Category | Driver |
|----------|--------|
| **Extraction** | Text File Extraction Driver |
| | XML File Extraction Driver |
| | Database Extraction Driver |
| | Isodaq File Extraction Driver |
| | Enviromon File Extraction Driver |
| **Inbound Connection** | File System Inbound Connection Driver |
| | FTP Inbound Connection Driver |
| | HTTP File Inbound Connection Driver |
| | Database Inbound Connection Driver |
| **Export** | AQUARIUS Time-Series Export Driver |
| **Outbound Connection** | HTTP Outbound Connection Driver |

#### Key rules & gotchas

- **`--restore` is destructive** — only use it for clean setups or full replacements
- **`settingOverrides` only work for keys listed in `ruleCanOverride`**; otherwise they are silently ignored
- **Rule profiles must exist before being referenced by name** in connectors; use `"ruleProfile": "Default"` with full `settingOverrides` when no shared profile is needed
- **Delta migration must be complete before activating Connect** — see Delta Migration section

### Key Configuration Steps

1. Identify all data sources and their connection types (FTP, hot folder, database, HTTP, etc.)
2. Map each data source feed to its target AQUARIUS time series (parameter ID + label + location identifier)
3. Author the provisioning JSON (locations → connectors → schedules → rule profiles)
4. Run the Provisioning Utility against the Connect server
5. Test ingestion with a small live data window before full activation
6. Confirm Connect-ingested data aligns with the end of the EXIM-imported historical data (no gap, no overlap)

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
