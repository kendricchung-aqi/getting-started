The ProvisioningTool **UnitGroup** task has not yet been fully documented here on this wiki.

Please refer to the relevant sample files included in the `ProvisioningTool.zip\SampleFiles\*` folder for example content:
- `SampleFiles\SampleCreateDeleteUnitGroups.csv`
- `SampleFiles\SampleUpdateUnitGroups.csv`

Alternatively, since the EXPORT operation is supported, you can run a `-Task="EXPORT UnitGroup all.unitgroups.csv"` operation to export all the current unit groups from your working AQTS app server, and use the generated CSV as a starting point for other CREATE, UPDATE, or DELETE operations.