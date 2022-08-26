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

## Format of the `UpdateLocationNotes.csv` file

The same CSV file format for the `-Task="CREATE LocationNote CreateLocationNotes.csv"` task can also be used for the `-Task="UPDATE LocationNote UpdateLocationNotes.csv"` task. But the CSV shape of an UPDATE task can be much thinner, only needing to supply one column to select a location, plus one column for each property to update.

The UPDATE LOCATION task needs a CSV with at least a LocationIdentifier or UniqueId column, plus any other columns of the location to be updated. Any columns not included in the CSV will not be modified.

Notes:
- You cannot update the `UtcOffset` column of an existing location. When the `-Task='UPDATE Location pathToCsv'` task is used, the `UtcOffset` column will be ignored if it exists in the CSV file.
- When all the column values match the current location's property values, no change will be made to the location. A location will only be modified when at least one property value is changed.

## Format of location note tag columns

The ProvisioningTool supports the setting of any configured tag. These values are identified by column names beginning with the `Tag:` prefix. The appropriate tag key name follows the colon.

Also assume these two Location tags have been configured (with `AppliesToLocations` applicability):

| CSV column header | Tag Key | Tag Type | Example value |
|---|---|---|---|
| `Tag:Watershed` | `Watershed` | `PickList`| `Fraser basin` |
| `Tag:Has Telemetry` | `Has Telemetry` | `None` | _any "Falsey" value_ - The tag will be removed.<br/>_any other value_ - The tag will be set.<br/><br/>See [below for details](#none-tag-type-boolean-values). |

### `None` tag type boolean values

The `None` tag type is treated as a slightly special case. These tags don't have any value. They are simply applied to an item (ie. enabled) or they are absent from an item (ie. disabled).

- **Disabled** for any of these seven "False-ish" value: blank/empty, `False`, `F`, `No`, `N`, `Off`, or `0` (case-insensitive).
- **Enabled** for any other value.

## Example CSV with extended attributes and tags

Here is an example CSV, with a header row and two location rows:

```csv
LocationIdentifier, LocationPath, LocationName, LocationType, UtcOffset, Description, Latitude, Longitude, Elevation, ElevationUnits, Publish, Ext:Province, Ext:Office, Ext:User, Ext:Status, Tag:Watershed, Tag:Has Telemetry
05JJ009, WSC.SASKATCHEWAN.REGINA, SALINE CREEK NEAR NOKOMIS, Hydrology Station, -06:00, The underpass near Hatfield Road., 51.41611, -105.10306, -105.10306, m, false, SASKATCHEWAN, REGINA, SUSAN.SMITH, ACTIVE, , YUP
08GA047, WSC.BRITISH COLUMBIA.NANAIMO, ROBERTS CREEK AT ROBERTS CREEK, Hydrology Station, -08:00, Where the bridge crosses the road, 49.42083, -123.64022, 15.3, m, false, BRITISH COLUMBIA, NANAIMO, FRANK.FROLLIC, ACTIVE, Fraser basin
```

Note that the `Yup` value for the `Tag:Has Telemetry` column could have been any non-False-ish value. Values of `Yes`, `1`, `OK`, or `I am a teapot` all have the same effect.

## Renaming an existing location identifier using the UPDATE task

An existing location can have its identifier changed using the `-Task="UPDATE Location pathToCsv"` task.

This section describes a few different ways a location can be renamed. The example CSVs have been trimmed down to the fewest required columns for clarity, but keep in mind that you can update all the attributes of a location with one CSV if you need to.

Three different columns supported by the UPDATE task will determine if a location's identifier will be updated:
- `LocationIdentifier`, `UniqueId`, and `UpdatedIdentifier`.
- When the `UniqueId` column exists and has a non-empty value, its value will be used to select the existing location to update.
- When the `UniqueId` column value is empty, or if the column doesn't exist in the CSV, then the `LocationIdentifier` column value will be used to select the existing location to update.

The above rules yield two different CSV shapes which can be used to rename a location:

### Option A) - Use `LocationIdentifier` and `UpdatedIdentifier`, but not `UniqueId`

This CSV shape is often the simplest for renaming, Just specify the old identifer and the new identifier.

```csv
LocationIdentifier, UpdatedIdentifier
05JJ009, 05JJ009_OLD
1100031, RenamedLoc
```

### Option B) - Use `UniqueId` and `LocationIdentifier`, but not `UpdatedIdentifier`

This CSV shape requires using the AQTS REST API to pull the list of locations and their uniqueID values. The `GET /AQUARIUS/Publish/v2/GetLocationDescriptionList` response has all the required pieces.

This shape is useful when you have already extracted the AQTS unique IDs and you know the new identifier values.

```csv
UniqueId, LocationIdentifier
b2c1b04f9e0547e7a23f9709137ddc7d, 05JJ009_OLD
729254bc35c84762a4fa829fa0f2f3ec, RenamedLoc
```

### Option C) - Use `UniqueId` and `UpdatedIdentifier`, ignoring `LocationIdentifier`

In this scenario, the rule of "`UpdatedIdentifier` trumps `LocationIdentifier`" kicks in. CSVs with this shape will be essentially ignoring the `LocationIdentifier` values completely, using the `UniqueId` to find the existing location, and then using the `UpdatedIdentifier` value as the location's new identifier.

```csv
UniqueId, LocationIdentifier, UpdatedIdentifier
b2c1b04f9e0547e7a23f9709137ddc7d, Can be anything, 05JJ009_OLD
729254bc35c84762a4fa829fa0f2f3ec, Does not matter, RenamedLoc
```
