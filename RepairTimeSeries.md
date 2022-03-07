# ProvisioningTool - `RepairTimeSeries` tasks

- Only the UPDATE operation is supported for this task.

### Database credentials are required for time-series repair

The `RepairTimeSeries` tasks require direct access to the AQTS database, since the Provisioning API doesn't support these types of changes.

See the [[Database Credentials]] topic for more details.

The simplest thing to do is run ProvisioningTool directly on the AQTS app server, to allow automatic discovery of the correct database configuration settings.

# The `RepairTimeSeries` operation is a bit dangerous. Proceed with caution!

The `RepairTimeSeries` operation allows some normally-locked properties of basic or reflected time-series to be changed.

This operation can be useful in the early stages of building a system, so that you can retain the points/corrections already in a series, without having to delete and reload everything.

But if you are regularly running the `RepairTimeSeries` operation as part of your organization's production workflow, then you're using it incorrectly and have bigger issues to solve.

# Intended use-cases for the `RepairTimeSeries` operation

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

If any of the CSV rows violate one of these constraints, then that invalid is logged and skipped.
