# ProvisioningTool - `LocationType` tasks

- CREATE, UPDATE, and DELETE operations are supported.

## Format of the `CreateLocationTypes.csv` file

| Column Header | Example value| Description |
|---|---|---|
| TypeName | `Sewage Station` | The text identifier for the location type. |
| Description | `It's nasty down here!` | Optional free-form description text. |
| UniqueId | `b2c1b04f9e0547e7a23f9709137ddc7d` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks. You can use either the `UniqueId` or `TypeName` column to specify the location type whose other properties will be updated. If both columns are used, `UniqueId` takes precedence. |
| UpdatedTypeName | `Sanitary Sewer Station` | This column is ignored in CREATE tasks, but can be used in UPDATE tasks to change the name of a location type when the column value is not an empty string. |
| Ext:*{key}* | `No` or `Yes` | Column headers beginning with "Ext:" followed by the key of the extended attribute can be used to assign an extended attribute value to a location type. The extended attribute must be configured with AppliesToLocationTypes = true. <br/><br/>This column represents a boolean value.<br/><br/>**Disabled** for blank/empty, `False`, `F`, `No`, `N`, `Off`, or `0` (case-insensitive).<br/>**Enabled** for any other value. |

## Format of the `UpdateLocationTypes.csv` file

The same CSV file format for the `-Task="CREATE LocationType CreateLocationTypes.csv"` task can also be used for the `-Task="UPDATE LocationType UpdateLocationTypes.csv"` task. But the CSV shape of an UPDATE task can be much thinner, only needing to supply one column to select a location type, plus one column for each property to update.

The UPDATE LOCATIONTYPE task needs a CSV with at least a TypeName or UniqueId column, plus any other columns of the location to be updated. Any columns not included in the CSV will not be modified.

## Format of the `DeleteLocationTypes.csv` file

The DELETE LOCATIONTYPE task needs a CSV with at least a TypeName or UniqueId column to identify the location type to delete. All other columns are ignored.
