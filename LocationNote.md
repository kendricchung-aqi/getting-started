# ProvisioningTool - `LocationNote` tasks

- EXPORT, CREATE, UPDATE, and DELETE operations are supported.

## Format of the `LocationNotes.csv` file

| Column Header | Example value| Description |
|---|---|---|
| UniqueId | `b2c1b04f9e0547e7a23f9709137ddc7d` | The unique ID for the note assigned by AQTS.<br/><br/>The `UniqueId` column is ignored for CREATE operations.<br/>The `UniqueId` column is optional for UPDATED operations.<br/>The `UniqueId` column is required for DELETE operations. |
| LocationIdentifier | `05JJ009` | The text identifier for the location note. |
| StartTime | `2022-04-15T13:45:00-07:00` | The optional start time of the note. |
| EndTime | `2022-04-15T15:45:00-07:00` | The optional end time of the note. |
| Details | 'The solar panel was stolen sometime after the 2021-Feb-18 field visit.' | A free form text field for the details of the note. Text containing commas or newlines must be quoted. |
| TimeSeries | `Stage.Telemetry@Loc1` | An optional time-series identifier for the note can be specified in any of three formats:<br/><br/>- `{Parameter}.{Label}` (eg. `Stage.Telemetry`)<br/>- `{Parameter}.{Label}@{Location}` (eg. `Stage.Telemetry@Loc1`)<br/>- `{TimeSeriesUniqueId}` (eg. `b2c1b04f9e0547e7a23f9709137ddc7d) |
| Tag:*{key}* |  | Column headers beginning with "Tag:" followed by the key of the tag can be used to assign a tag value to a location note. The tag must be configured with AppliesToLocationNotes = true in order to set a value for the location note. |

## UPDATE operations

The same CSV file format for the `-Task="CREATE LocationNote CreateLocationNotes.csv"` task can also be used for the `-Task="UPDATE LocationNote UpdateLocationNotes.csv"` task. But the CSV shape of an UPDATE task can be thinner, with only the columns required to identify the note and the properties which are changing. Properties not included in the CSV column header will not be updated.

The best way to identify a location note is through its `UniqueId` property, available from the `GET AQUARIUS/Publish/v2/GetLocationData` API response, or by running the EXPORT operation.

If no `UniqueId` value exists (or of no `Unique` column header exists for the whole file), but a note has a `StartTime` and `EndTime`, then the note can be identified through the combination of the `LocationIdentifier`, `StartTime`, and the `EndTime` columns.

Notes with no start time or no end time can only be identified using the `UniqueId` column.

## DELETE operations

Only the `UniqueId` column is used, so it must be present. All other columns will be ignored.

## Format of location note tag columns

The ProvisioningTool supports the setting of any configured tag with `AppliesToLocationNotes` applicability. These tag values are identified by column names beginning with the `Tag:` prefix. The appropriate tag key name follows the colon.

The same [`Tag:xxx` format and rules used by the Location task](https://github.com/AquaticInformatics/getting-started/wiki/Location#format-of-location-tag-columns-and-location-extended-attribute-columns) is used to specify location note tags.
