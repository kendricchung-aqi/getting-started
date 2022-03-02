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

| Task | File format | Create | Update | Delete |
| --- | --- | --- | --- | --- |
| Parameter                      | csv, xls | Y | Y | N |
| Unit                           | csv, xls | Y | Y | N |
| UnitGroup                      | csv      | Y | Y | Y |
| MonitoringMethod               | csv      | Y | Y | Y |
| [[Location]]                       | csv, xls | Y | Y | N |
| TimeSeries                     | csv, xls | Y | Y | N |
| Grade                          | csv, xls | Y | Y | Y |
| Role                           | csv      | Y | Y | Y |
| FolderUserRole                 | csv      | Y | Y | Y |
| LocationUserRole               | csv      | Y | Y | Y |
| PrimaryFolder                  | csv      | Y | Y | Y |
| TimeSeriesExtendedAttributes   | csv, xls | Y | Y | N |
| SecondaryFolder                | csv      | Y | N | N |
| Approval                       | csv, xls | Y | Y | Y |
| GlobalSetting                  | csv      | Y | Y | Y |
| LocationNote                   | csv      | Y | Y | Y |
| Tag                            | csv      | Y | Y | Y |
| StandardDatum                  | csv      | Y | N | Y |
| ReferencePoint                 | csv      | Y | Y | Y |
| ReferencePointPeriod           | csv      | Y | Y | N |
| LocationStandardDatum          | csv      | Y | Y | Y |
| LocalAssumedDatumPeriod        | csv      | Y | Y | Y |
| DatumReading                   | csv      | N | Y | N |
| MeasurementGrade               | csv      | N | Y | N |
| PicklistDisplayItem            | csv      | Y | Y | Y |
| Sensor                         | csv      | Y | Y | Y |
| RepairTimeSeries               | csv      | N | Y | N |
| ExtendedAttributeSchema        | sql      | Y | N | N |

# Sample files

Check the included `SampleFiles` for item-specific format examples.

# Configuring your AQTS picklists

The `PicklistDisplayItem` tasks are used to configure the dropdown menus visible in the AQTS system (typically in the browser web forms).

These tasks configure the `PicklistDisplayItem` database table, which stores a localizable list of values.

The AQUARIUS browser apps (Springboard, Field Visit, Location Manager) try to use the current language definition for a list item, falling back to the English item if no language-specific item exists.

### Database credentials are required for picklist and extended attribute schema configuration

The `PicklistDisplayItem` and `ExtendedAttributeSchema` tasks require direct access to the AQTS database.

The `/DbFilename=`, `/DbType=`, and `/DbConnectionString=` command-line options allow you to explicitly provide database credentials when they cannot be inferred from the `/Server=` context.

If you are running `ProvisioningTool.exe` directly on the AQTS app server, or if the `\\server\C$\ProgramData\Aquatic Informatics\AQUARIUS\AquariusDataSource.xml` file is readable over a network share, then the tool should be able to automatically infer the DB credentials for you.

Try to use the automatically inferred DB credentials before manually setting the `/DbFilename=`, `/DbType=`, or `/DbConnectionString=` options.

### Fixed-size vs. Free-form picklists

Picklists fall into one of two categories:
- **Freeform lists**, where you are free to define as many or as few items as you'd like.
- **Fixed-sized lists**, where the number of items is fixed. You can change the display names, or add translated versions, but if AQTS is expecting a list of 10 items, your customization must still include 10 items.

Most picklists in the system are fixed-sized lists. The lists can change from release to release, so please contact our Support Team to help guide you through this configuration process.

### AQTS 2018.4 free-form lists

The 2018.4 free-form lists are these `PicklistKey` values:

```
ConditionType
ControlType
DriftCheckType
FlowOverControlType
IceAssemblyType
LevelSurveyMethod
ReadingQualifierType
SuspensionWeightType
ThresholdName
VelocityObservationMethodType
ViewModeType
```

All other picklists are fixed sized lists.

# Help page (via the `-help` option)

```
Purpose: Set up system codes,locations,time series etc. on an AQTS 201x server.

Usage: ProvisioningTool [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  ==================== Server credentials (required for all tasks)
  -Server              The AQTS 201x app server you want to configure.
  -Username            AQTS username. [default: admin]
  -Password            AQTS password. [default: admin]

  ==================== Database credentials (only required for PicklistDisplayItem, ExtendedAttributeSchema, PanelCount tasks)
  -DbFilename          The filename of database configuration file.
  -DbType              Override the database type. One of Unknown, MsSql, Oracle, Postgres. [default: use the DbFilename.]
  -DbConnectionString  Override the DB connection string [default: use the DbFilename.]

  ==================== Task options
  -Task                The setup task to perform. Use quotation marks. E.g., Task="create grade c:\Input\grades.csv"
  -SkipConfirmation    Set to true to confirm tasks to be performed. [default: False]

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.

Valid Operations:
    Create
    Update
    Delete.

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
    TimeSeriesExtendedAttributes
    SecondaryFolder
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
    RepairTimeSeries.
```