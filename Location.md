# ProvisioningTool - `Location` tasks

- EXPORT, CREATE, and UPDATE operations are supported.
- DELETE operations are not supported for locations. But you can use the [LocationDeleter.exe tool](https://github.com/AquaticInformatics/examples/tree/master/TimeSeries/PublicApis/SdkExamples/LocationDeleter#locationdeleter) for that.

## Format of the `CreateLocations.csv` file

| Column Header | Example value| Description |
|---|---|---|
| LocationIdentifier | `05JJ009` | The text identifier for the location. |
| LocationPath | `WSC.SASKATCHEWAN.REGINA` | The folder path of the location |
| LocationName | `SALINE CREEK NEAR NOKOMIS` | A longer name of the location. |
| LocationType | `Hydrology Station` | The type of location |
| UtcOffset | `-06:00` | The offset from UTC, in +HH:MM or -HH:MM format.|
| Description | `The underpass near Hatfield Road.` | A long description of the location |
| Latitude | `51.41611` | WGS84 latitude. |
| Longitude | `-105.10306` | WGS84 longitude. |
| Elevation | `504.5` | Elevation of the location. |
| ElevationUnits | `m` | Units for elevation. |
| Publish | `false` | A `true` or `false` flag controlling the publish status of the location. |
| UniqueId | `b2c1b04f9e0547e7a23f9709137ddc7d` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks. You can use either the `UniqueId` or `LocationIdentifier` column to specify the location whose other properties will be updated. If both columns are used, `UniqueId` takes precedence. |
| UpdatedIdentifier | `05JJ009_OLD` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks to change the identifier text of a location when the column value is not an empty string. |
| Tag:*{key}* |  | Column headers beginning with "Tag:" followed by the key of the tag can be used to assign a tag value to a location. The tag must be configured with AppliesToLocations = true in order to set a value for the location. |
| Ext:*{key}* | `REGINA` for `Ext:Office` | Column headers beginning with "Ext:" followed by the key of the extended attribute can be used to assign an extended attribute value to a location. The extended attribute must be configured with AppliesToLocations = true or AppliesToLocationTypes = true in order to set a value for the location. |

## Format of the `UpdateLocations.csv` file

The same CSV file format for the `-Task="CREATE Location CreateLocations.csv"` task can also be used for the `-Task="UPDATE Location UpdateLocations.csv"` task. But the CSV shape of an UPDATE task can be much thinner, only needing to supply one column to select a location, plus one column for each property to update.

The UPDATE LOCATION task needs a CSV with at least a LocationIdentifier or UniqueId column, plus any other columns of the location to be updated. Any columns not included in the CSV will not be modified.

Notes:
- For AQTS `2025.1` and later, you can use `ProvisioningTool V3.0.434+` to update `UTCOffset` of a location.
- For AQTS earlier than `2025.1`, you cannot update the `UtcOffset` column of an existing location. When the `-Task='UPDATE Location pathToCsv'` task is used, the `UtcOffset` column will be ignored if it exists in the CSV file.
- When all the column values match the current location's property values, no change will be made to the location. A location will only be modified when at least one property value is changed.

## Format of location tag columns and location extended attribute columns

The ProvisioningTool supports the setting of any configured tag or extended attribute values. These values are identified by column names beginning with the `Tag:` or `Ext:` prefix. The appropriate tag or extended attribute key name follows the colon.

Assume there are four extended attributes which can be set on a location:

| CSV column header | Extended attribute display name | Example value |
|---|---|---|
| `Ext:Province` | `Province` | `SASKATCHEWAN` |
| `Ext:Office` | `Office` | `REGINA` |
| `Ext:User` | `User` | `SUSAN.SMITH` |
| `Ext:Status` | `Status` | `ACTIVE` |

Also assume these two Location tags have been configured (with `AppliesToLocations` applicability):

| CSV column header | Tag Key | Tag Type | Example value |
|---|---|---|---|
| `Tag:Watershed` | `Watershed` | `PickList`| `Fraser basin` |
| `Tag:Has Telemetry` | `Has Telemetry` | `None` | _any "Falsey" value_ - The tag will be removed.<br/>_any other value_ - The tag will be set.<br/><br/>See [below for details](#none-tag-type-boolean-values). |

### `None` tag type boolean values

The `None` tag type is treated as a slightly special case. These tags don't have any value. They are simply applied to an item (ie. enabled) or they are absent from an item (ie. disabled).

- **Disabled** for any of these seven "False-ish" value: blank/empty, `False`, `F`, `No`, `N`, `Off`, or `0` (case-insensitive).
- **Enabled** for any other value.

### `Picklist` tag type values

Picklists allow a value to be selected from a configured list. Both tags and extended attributes can be `Picklist` types, but tags can have more than one active value from the list, whereas extended attributes can only have one value, or no value at all.

Consider a picklist called `BestBeetle`, with the values `John`, `Paul`, `George`, and `Ringo`.

When a picklist tag has more than one value, the values are stored in the CSV in a comma separated list and the entire list of values must be contained in double quotes.

A location tag `Tag:BestBeetle` might have a value of `"John,Paul"` but an extended attribute `Ext:BestBeetle` could only choose one value.

(And in a sane universe, that value **should** only ever be `Ringo`, but that is a debate for another day.)

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
