# ProvisioningTool - `RepairTimeSeries` task

### Read this before you start:

- Read the "WARNING" and "Intended use cases" sections below and decide if this task is safe and right for solving your problems.
- `RepairTimeSeries` will make changes even if the time series is locked.
- You won't see any snapshots, audit history of the changes made by this task.
- Changing the default grade, interpolation type will affect the existing data immediately. Any non-default grades on the data points will remain, but the default grades will be changed to the new one that you just specified. 
- If you have have generated reports, you may want to regenerate them after running this task.

### Quick start 

- Prepare a csv file. See below or comments in the sample file for instructions.
- Run this command: `ProvisioningTool.exe /Server=<My_AQTS_Server_Url_Or_Host> /UserName=<MyUser> /Password=<MyPassword> /task="Update RepairTimeSeries Path\to\my_repair_ts.csv"`

### Database credentials are required for time-series repair

The `RepairTimeSeries` tasks require direct access to the AQTS database, since the Provisioning API doesn't support these types of changes.

See the [[Database Credentials]] topic for more details.

The simplest thing to do is run ProvisioningTool directly on the AQTS app server, to allow automatic discovery of the correct database configuration settings.

# WARNING: The `RepairTimeSeries` operation is a bit dangerous. Proceed with caution!

The `RepairTimeSeries` operation allows some normally-locked properties of basic or reflected time-series to be changed.

This operation can be useful in the early stages of building a system, so that you can retain the points/corrections already in a series, without having to delete and reload everything.

But if you are regularly running the `RepairTimeSeries` operation as part of your organization's production workflow, then you're using it incorrectly and have bigger issues to solve.

# Intended use cases for the `RepairTimeSeries` operation

- Use a simple CSV file to describe which properties of a series should be changed.
- Change from one parameter to another
- Change from one unit to another within the parameter's unit group
- Move the series to another location
- Change the default grade code (either by numeric code or by grade name)
- Change the interpolation type
- Any combination of the above.

All the changes are made quickly in the DB, while retaining the current points, corrections, and approvals.

# Caveats

The `RepairTimeSeries` operation enforces/validates quite a few preconditions, to prevent a user from creating an invalid time-series that otherwise the product would not allow:

- The location/parameter/label uniqueness constraint is maintained.
- You can only repair basic or reflected series. You can't repair a derived series (maybe in a future release, but that's waaay more complex)
- You can't repair a series if it is an input to a derived processing plan (for the same reasons. PROC validation is complex)
- You can't change parameters if the series has any parameter-specific method codes.
- You can't change interpolation to InstantaneousTotals (Type 6) or DiscreteValues (Type 7) if the series has any non MaxGap tolerances set.

If any of the CSV rows violate one of these constraints, then they will be marked as invalid and skipped. Invalid rows will be saved to a csv file when the task is completed.

## CSV columns

| Column header | Description |
|---|---|
| TimeSeriesUniqueId | The unique 32-character identifier for the series.<br/><br/>Either `TimeSeriesUniqueId ` or all three of `LocationIdentifier`, `ParameterId`, and `UnitId` must be set. |
| LocationIdentifier | The location identifier of the series.<br/><br/>Either `TimeSeriesUniqueId ` or all three of `LocationIdentifier`, `ParameterId`, and `UnitId` must be set. |
| ParameterId | The parameter ID of the series (`HG` not `Stage`).<br/><br/>Either `TimeSeriesUniqueId ` or all three of `LocationIdentifier`, `ParameterId`, and `UnitId` must be set. |
| Label | The label of the series.<br/><br/>Either `TimeSeriesUniqueId ` or all three of `LocationIdentifier`, `ParameterId`, and `UnitId` must be set. |
| UnitId | If set, the unit ID will be updated. The unit ID must be part of the parameter's unit group. |
| InterpolationType | If set, must be one of:<br/><br/>- `InstantaneousValues` (Type 1)<br/>- `PrecedingConstant` (Type 2)<br/>- `PrecedingTotals` (Type 5)<br/>- `InstantaneousTotals` (Type 6)<br/>- `DiscreteValues` (Type 7)<br/>- `SucceedingConstant` (Type 8) |
| DefaultGradeCodeOrName | If set, the default grade stripe for the series will be updated. This value can be a numeric grade code like `10` or a grade name like `Good`. |
| UpdatedLocationIdentifier | If set, this will be the identifier of the location that now owns the series. |
| UpdatedParameterId | If set, the parameter ID will be changed to this value. |
| UpdatedLabel | If set, the label will be changed to this value. |

### Sample CSV file

```csv
# Repairing a time-series is a special UPDATE task to change some normally-unchangeable properties of time-series, while retaining the rest of the existing points, corrections, and other metadata.
# CREATE and DELETE tasks for RepairTimeSeries are not supported.
#
# Please use with caution and ONLY after verifying that your database and blob storage backup & restore procedures are working.
#
# An existing time-series can be identified by one of two methods:
# 1) Just the TimeSeriesUniqueId column (1st column).
# 2) The combination of LocationIdentifier, ParameterId, and Label (the next 3 columns), when column 1 is empty.
#
# When the TimeSeriesUniqueId column is set, then the LocationIdentifier, ParameterId, and Label columns can be set to their new desired values.
#
# Other optional columns:
# - UnitId - If set, must be one of the unit IDs in the target parameter ID's unit group
# - InterpolationType - If set, must be one of: InstantaneousValues, PrecedingConstant, PrecedingTotals, InstantaneousTotals, DiscreteValues, SucceedingConstant
# - DefaultGradeCodeOrName - If set, must be either a numeric grade code or name. The series default grade code will be set to match.
# - UpdatedLocationIdentifier - If set, will update the location identifier.
# - UpdatedParameterId - If set, will update the parameterId.
# - UpdatedLabel - If set, will update the label.
#
# Types of repair actions which are possible:
# - Switch the series to another parameter
# - Switch ownership to another location
# - Change the unit
# - Change the default grade code (which applies to newly appended points).
# - Change the interpolation type
# - Change the label (sometimes required to maintain the location/parameter/label uniqueness constraint)
# Any combination of the above

# Constraints:
# Rows with invalid values will be skipped (for any invalid location identifiers, parameter IDs, unit IDs, grade codes, interpolation types)
# You can only repair Basic and Reflected series, not Derived series.
# You cannot repair a series if it is used as an input to a derivation processing plan.
# You cannot switch a series to a different parameter if the series has parameter-specific method codes.
# An updated unit ID must be within the target parameter's unit group.
# Parameters are referenced by ID, not identifier. So "HG" but not "Stage".
# If changing the parameter and/or location, you cannot violate the location/parameter/label uniqueness constraint. You may need to set an updated label to avoid this.

# General guidance:
# Only specify the columns you actually need to change. Leave other columns blank/empty, to indicate these are not being changed.

# This 10-column header line must be the first non-blonk, non-commented line in the CSV
TimeSeriesUniqueId, LocationIdentifier, ParameterId, Label, UnitId, InterpolationType, DefaultGradeCodeOrName, UpdatedLocationIdentifier, UpdatedParameterId, UpdatedLabel

# The next row is an example how you can reference a series by its unique ID and move it to another location. Only the first 2 columns are required.
a4ee555ecd4a4e0ca5283fb8a6bb7e06, NewLocation

# The next row changes the parameter to a Stage series in meters and the default grade to Excellent and leaving everything else unchanged
870e9bd1129d438da6dfe3e782ca3bb9, , HG, , m, , Excellent

# The next row uses the location/parameter/label to identify the series. The series is changed to a Depth series with a new label, a default grade code of 11, and a change of interpolation.
, 5016, HG, Historic, ft, InstantaneousValues, 11, , Depth, FromTopOfCasing
```
(Page updated: 2026-March-18th)