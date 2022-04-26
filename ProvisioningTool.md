Download the [latest ProvisioningTool.zip here](https://github.com/AquaticInformatics/getting-started/releases/ProvisioningTool)

# ProvisioningTool.exe

This console utility allows you to provision/configure a number of AQTS settings, using CSV data files.

This tool supports the flexible [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) for when the command line gets a bit daunting.

The ProvisioningTool allows you to perform bulk provisioning operations on various types of AQUARIUS items for AQUARIUS Time-Series systems running 2018.3-or-newer.

## Installation

- Download the `ProvisioningTool.zip` archive
- Extract the Zip archive to a new folder
- See the `SampleFiles` subfolder for examples of different CSV data files.
- Open a CMD.EXE, bash shell, or Powershell window to the folder, and run the commands.

# Running a provisioning task

The tool can accept multiple `/task="operation item filename"` options. The tool will perform the operations in the order in which they are specified.

```cmd
ProvisioningTool.exe /Server=myserver /Task="create parameter ../../mynewparameters.csv" /Task="update parameter ../../myupdatedparameters.csv"
```

## Use the `@options.txt` syntax to simplify repetitive tasks.

The [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) can be a very helpful way to simplify a number of repetitive provisioning tasks.

You can use the `#` or `//` prefix to add comment lines, you can leave blank lines, and you don't need to worry about shell-specific quoting rules.

The above command line simplifies down to this:

```cmd
ProvisioningTool.exe @mytasks.txt
```

With the `mytasks.txt` file as the following 6 lines:

```sh
# Set the credentials here
-Server=myserver

# Perform all these tasks, in this order
-task=Create Parameter ../../myNewParameters.csv
-task=Update parameter ../../MyUpdatedParameters.csv
```

# Supported Provisioning Operations

Click on the **Task** column to get detailed information for a specific task.

(Any task listed in red has not yet been documented in the wiki, so try an `EXPORT` operation on an existing system if supported, or consult the appropriate sample files)

| Task | File format | Create | Update | Delete | Export | 
| --- | --- | --- | --- | --- | --- |
| [[Parameter]]                      | csv, xls | Y | Y | Y | Y
| [[Unit]]                           | csv, xls | Y | Y | Y | Y
| [[UnitGroup]]                      | csv, xls | Y | Y | Y | Y
| [[MonitoringMethod]]               | csv, xls | Y | Y | Y | Y
| [[LocationType]]                   | csv, xls | Y | Y | Y | Y
| [[Location]]                       | csv, xls | Y | Y | N | Y
| [[TimeSeries]]                     | csv, xls | Y | Y | N | Y
| [[Grade]]                          | csv, xls | Y | Y | Y | Y
| [[Qualifier]]                      | csv, xls | Y | Y | Y | Y
| [[ApprovalLevel]]                  | csv, xls | Y | Y | Y | Y
| [[Tag]]                            | csv, xls | Y | Y | Y | Y
| [[ExtendedAttribute]]              | csv, xls | Y | Y | Y | Y
| [[GlobalSetting]]                  | csv, xls | Y | Y | Y | Y
| [[Role]]                           | csv      | Y | Y | Y |
| [[FolderUserRole]]                 | csv      | Y | Y | Y |
| [[LocationUserRole]]               | csv      | Y | Y | Y |
| [[PrimaryFolder]]                  | csv      | Y | Y | Y |
| [[SecondaryFolder]]                | csv      | Y | N | N |
| [[LocationNote]]                   | csv      | Y | Y | Y |
| [[StandardDatum]]                  | csv      | Y | N | Y |
| [[ReferencePoint]]                 | csv      | Y | Y | Y |
| [[ReferencePointPeriod]]           | csv      | Y | Y | N |
| [[LocationStandardDatum]]          | csv      | Y | Y | Y |
| [[LocalAssumedDatumPeriod]]        | csv      | Y | Y | Y |
| [[DatumReading]]                   | csv      | N | Y | N |
| [[MeasurementGrade]]               | csv      | N | Y | N |
| [[PicklistDisplayItem]]            | csv      | Y | Y | Y |
| [[Sensor]]                         | csv      | Y | Y | Y |
| [[RepairTimeSeries]]               | csv      | N | Y | N |
| [[ExtendedAttributeSchema]]        | sql      | Y | Y | N |

# Sample files

Check the included `SampleFiles` folder for item-specific format examples.

# Help page (via the `-help` option)

```
Purpose: Set up system codes,locations,time series etc. on an AQTS 20xx server.

Usage: ProvisioningTool [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  ==================== Server credentials (required for all tasks)
  -Server              The AQTS 20xx app server you want to configure.
  -Username            AQTS username. [default: admin]
  -Password            AQTS password. [default: admin]

  ==================== Database credentials (only required for PicklistDisplayItem, ExtendedAttributeSchema, PanelCount, RepairTimeSeries tasks)
  -DbFilename          The filename of database configuration file.
  -DbType              Override the database type. One of Unknown, MsSql, Postgres. [default: use the DbFilename.]
  -DbConnectionString  Override the DB connection string [default: use the DbFilename.]

  ==================== Task options
  -Task                The setup task to perform. Use quotation marks. E.g., Task="create grade c:\Input\grades.csv"
  -SkipConfirmation    Set to true to confirm tasks to be performed. [default: False]
  -AllowOverwrite      When true, an Export operation can overwrite an existing file. [default: False]

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.

Valid Operations:
    Create
    Update
    Delete
    Export.

Supported Data names:
    Parameter
    Location
    TimeSeries
    Unit
    Grade
    Role
    FolderUserRole
    UnitGroup
    PrimaryFolder
    SecondaryFolder
    LocationType
    Approval
    GlobalSetting
    PicklistDisplayItem
    LocationUserRole
    MonitoringMethod
    LocationNote
    ExtendedAttributeSchema
    QualitativeUncertainty
    MeasurementGrade
    PanelCount
    ReferencePoint
    ReferencePointPeriod
    Tag
    StandardDatum
    LocationStandardDatum
    LocalAssumedDatumPeriod
    DatumReading
    Sensor
    RepairTimeSeries
    Qualifier.
```