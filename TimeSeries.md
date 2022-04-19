# ProvisioningTool - `TimeSeries` tasks

- CREATE and UPDATE operations are supported.
- DELETE operations are not supported for time-series. But you can use the [LocationDeleter.exe tool](https://github.com/AquaticInformatics/examples/tree/master/TimeSeries/PublicApis/SdkExamples/LocationDeleter#locationdeleter) for that.

## Format of the `CreateTimeSeries.csv` file

| Column Header | Example value| Description |
|---|---|---|
| LocationIdentifier | `05JJ009` | The text identifier for the location owning the series. |
| ParameterId | `HG` | The parameter ID for the series (use the ID "HG", not the Display ID "Stage"). |
| Label | `Telemetry` | Any non-empty string. Labels must be unique for each location/parameterID. | 
| TimeSeriesType| One of: <br/> `Basic` <br/> `Reflected` | The type of series. |
| UnitId | `ft` | The unit ID of the series. Can be any unit within the parameter's unit group. |
| InterpolationType | One of: <br/> `InstantaneousValues` <br/> `PrecedingConstant` <br/> `PrecedingTotals` <br/> `InstantaneousTotals` <br/> `DiscreteValues` <br/> `SucceedingConstant` | The interpolation type of the series. |
| UtcOffset | `-06:00` | The offset from UTC, in +HH:MM or -HH:MM format. Defaults to the location's UTC offset if blank or omitted. |
| GapToleranceInMinutes | `60` | The initial gap tolerance, in minutes, for the series. |
| Description | `Bubbler hose under the gage house.` | A description of the series. Can be blank or omitted. |
| Comment | `Watch out for frogs!` | A comment for the series. Can be blank or omitted. |
| Method | `HGLOGGER` | The method code for the parameter. See the System Config **Methods** page for details. <br/>Defaults to `DefaultNone` if blank or omitted. |
| Publish | `false` | A `true` or `false` flag controlling the publish status of the series. |
| SubLocationIdentifier | `GageHouse` | The sub-location identifier within the location. Can be blank or omitted. |
| ComputationIdentifier | One of: <br/> `Min` <br/> `Max` <br/> `Sum` <br/> `Mean` <br/> `Median` <br/> `Selected Value` <br/> `Tidal High` <br/> `Tidal Lower High` <br/> `Tidal Higher Low` <br/> `Tidal Low` <br/> `Decumulated` <br/> `Max At Event Time` <br/> `Total Amount` | The identifier of the computation type. See the System Config **Computation Types** page for details. <br/>Can be blank or omitted. |
| ComputationPeriodIdentifier | One of: <br/> `Annual` <br/> `Monthly` <br/> `Weekly` <br/> `Daily` <br/> `Hourly` <br/> `Minutes` <br/> `Points` <br/> `Water Year` | The identifier of the computation period. See the System Config **Computation Periods** page for details. <br/>Can be blank or omitted. |
| UniqueId | `b2c1b04f9e0547e7a23f9709137ddc7d` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks. You can use either the `UniqueId` column or all three of the `LocationIdentifier`, `ParameterId`, and `Label` columns to identify the series whose other properties will be updated. If all 4 columns are used, `UniqueId` takes precedence. |
| UpdatedLabel | `Telemtry_OLD` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks to change the label text of a series when the column value is not an empty string. |
| Ext:*{key}* | `Vedas` for `Ext:Logger` | Column headers beginning with "Ext:" followed by the key of the extended attribute can be used to assign an extended attribute value to a series. The extended attribute must be configured with AppliesToTimeSeries = true in order to set a value for the series. |

## Format of the `UpdateTimeSeries.csv` file

The same CSV file format for the `-Task="CREATE TimeSeries CreateTimeSeriess.csv"` task can also be used for the `-Task="UPDATE TimeSeries UpdateTimeSeries.csv"` task. But the CSV shape of an UPDATE task can be much thinner, only needing to supply enough columns to select a series (just the `UniqueId` column or all three `LocationIdentifier`, `ParmeterId`, and `Label` columns), plus one column for each property to update. Any columns not included in the CSV will not be modified.

Notes:
- You cannot update the `UtcOffset` column of an existing series. When the `-Task='UPDATE TimeSeries pathToCsv'` task is used, the `UtcOffset` column will be ignored if it exists in the CSV file.
- When all the column values match the current series' property values, no change will be made to the series. A series will only be modified when at least one property value is changed.

## Format of time-series extended attribute columns

The ProvisioningTool supports the setting of any configured extended attribute values. These values are identified by column names beginning with the `Ext:` prefix. The appropriate extended attribute key name follows the colon.

Assume there are two extended attributes which can be set on a series:

| CSV column header | Extended attribute display name | Example value |
|---|---|---|
| `Ext:Logger` | `Logger` | `Vedas II` |
| `Ext:Status` | `Status` | `ACTIVE` |

## Example CSV with extended attributes

Here is an example CSV, with a header row and two series rows:

```csv
LocationIdentifier, ParameterId, Label, TimeSeriesType, UnitId, InterpolationType, UtcOffset, GapToleranceInMinutes, Description, Comment, Publish, Method, ComputationIdentifier, ComputationPeriodIdentifier, SubLocationIdentifier, UniqueId, UpdatedLabel, Ext:Logger, Ext:Status
Loc1, HG, Telemetry, Basic, ft, InstantaneousValues, , 60, Bubbler hose under the gage house., Watch out for frogs!, false, HGLOGGER, , , , , , Vedas II, Active
Loc1, PP, External, Reflected, in, PrecedingTotals, , 60, From weather agency, , false, , , , , , , , Active
```

## Renaming an existing series using the UPDATE task

An existing series can have its label changed using the `-Task="UPDATE TimeSeries pathToCsv"` task.

This section describes a few different ways a series can be renamed. The example CSVs have been trimmed down to the fewest required columns for clarity, but keep in mind that you can update all the attributes of a series with one CSV if you need to.

Five different columns supported by the UPDATE task will determine if a series' will be renamed:
- `LocationIdentifier`, `ParameterId`, `Label`, `UniqueId`, and `UpdatedLabel`.
- When the `UniqueId` column exists and has a non-empty value, its value will be used to select the existing series to update.
- When the `UniqueId` column value is empty, or if the column doesn't exist in the CSV, then the `LocationIdentifier` and `ParameterId` and `Label` column values will be used to select the existing series to update.

The above rules yield different CSV shapes which can be used to rename a series:

### Option A) - Use `LocationIdentifier`+`ParameterId`+`Label` and `UpdatedLabel`, but not `UniqueId`

This CSV shape is often the simplest for renaming, since you can work with the text identifiers you already know. Just specify the old identifer and the new identifier.

```csv
LocationIdentifier, ParameterId, Label, UpdatedLabel
05JJ009, HG, Telemetry, Telemetry_OLD
Loc1, PP, External, OtherAgency
```

### Option B) - Use `UniqueId` and `Label`, but not `UpdatedLabel`

This CSV shape requires using the AQTS REST API to pull the list of locations and their uniqueID values. The `GET /AQUARIUS/Publish/v2/GetTimeSeriesDescriptionList` response has all the required pieces.

This shape is useful when you have already extracted the AQTS unique IDs and you know the new label values.

```csv
UniqueId, Label
b2c1b04f9e0547e7a23f9709137ddc7d, Telemetry_OLD
729254bc35c84762a4fa829fa0f2f3ec, OtherAgency
```

### Option C) - Use `UniqueId` and `UpdatedLabel`, ignoring `Label`

In this scenario, the rule of "`UpdatedLabel` trumps `Label`" kicks in. CSVs with this shape will be essentially ignoring the `Label` values completely, using the `UniqueId` to find the existing series, and then using the `UpdatedLabel` value as the series' new label.

```csv
UniqueId, Label, UpdatedLabel
b2c1b04f9e0547e7a23f9709137ddc7d, Can be anything, Telemtry_OLD
729254bc35c84762a4fa829fa0f2f3ec, Does not matter, OtherAgency
```
