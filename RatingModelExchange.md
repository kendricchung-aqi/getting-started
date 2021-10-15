Download the [latest RatingModelEchange.exe here](https://github.com/AquaticInformatics/getting-started/releases/RatingModelExchange)

## RatingModelExchange

- A single EXE .NET console utility for provisioning new rating models.
- Reads rating model definitions from a CSV file.
- Imports rating models to an AQTS system and/or saves a HydroML file to use for later import steps.
- Can also export one or more rating models from an AQTS server to a HydroML file and CSV file.
- This tool supports the flexible [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) for when the command line gets a bit daunting.

## Import to AQTS and/or save a HydroML file

- When the `/Server=hostname` option is specified, the models defined in the CSV will be imported to the AQTS system.
- When the `/SaveHydroML=file` or `/SaveRoot=dir` options are set, the equivalent HydroML XML and CSV files will be saved.
- When an existing directory is given as the `/SaveHydroML=dir` argument, the HydroML and CSV files will be saved in the directory, using the rating model identifier as the filename.
- You can combine both saving and importing options.
- Existing XML files will not be overwritten unless the `/Overwrite=true` option is set.

### Saving ratings in TimeSeriesImporter format

The `/SaveRoot=dir` option can save the rating models in the file structure expected by the `TimeSeriesImporter.exe` tool, which can be useful for migrating systems into AQTS.

The root folder specified must already exist, but the tool will create any necessary subfolders.

## Export existing rating models

The tool can also export one or more rating models to a HydroML and/or CSV file.

The `/Export={ratingModelIdentifer}` option can be set multiple times to export more than one rating model in a single pass.

You can also omit the `/Export=` prefix from the command line and just specify the rating model identifier as a positional argument. The tool will recognize the `{Input}-{Output}.{Label}@{Location}` syntax and save you a bit of typing.

This shorter command line:
```
$ RatingModelExchange -server=doug-vm2012r2 -saveHydroML=output Stage-Discharge.Primary@Loc1 Stage-Discharge.Primary@Loc2
```

is equivalent to this longer command line:
```
$ RatingModelExchange -server=doug-vm2012r2 -saveHydroML=output -export=Stage-Discharge.Primary@Loc1 -export=Stage-Discharge.Primary@Loc2
```

If you try to export a rating model, you will need to specify one of the `/SaveHydroML=fileOrDir` or `/SaveRoot=dir` options so that the tool knows where to save the exported files.

## Updating existing rating models

By default, attempts to update existing models on the AQTS server will prevented. This makes `RatingModelExchange` safe to use on most AQTS systems without causing disruption.

Two options will allow you to override existing rating models, which can be useful to retain existing rating derivation chains, but make updates to the models used by derivations.

- `/UpdateExistingModels=true` will allow an existing rating model to be updated.
- `/DeleteExistingCurves=true` will delete all current curves within a model, before merging in the curves defined by the CSV rows.

When `/UpdateExistingModels=true` and `/DeleteExistingCurves=false`, any existing curves that are not included in the CSV rows will be retained by their model.

## CSV file format

- Blank rows are skipped.
- Rows starting with `#` or `//` are treated as comments and are skipped.
- Leading and trailing whitespace is trimmed from each field.
- Quotes are not required for string fields, unless the field contains a comma

8 row types are supported. The first field in each row defines the row type.

| Row type | Description |
| --- | --- |
| [`Model`](#model-rows) | Creates a new rating model. |
| [`Curve`](#curve-rows) | Creates a new rating curve in the current rating model. |
| [`Period`](#period-rows) | Creates a new period of applicability for a rating curve in the current rating model. |
| [`TablePoint`](#tablepoint-rows) | Adds a descriptor point to the current rating curve. |
| [`Formula`](#formula-rows) | Generates an equivalent table from an algebraic formula. |
| [`Shift`](#shift-rows) | Adds a shift to the current rating curve. |
| [`Offset`](#offset-rows) | Sets the offset of the current rating curve. |
| [`Grade`](#grade-rows) | Adds a grade mapping to the current rating curve. |

See [Timestamp Formats](#timestamp-formats) for the supported date/time formats

### `Model` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Model`. |
| 2 | Location | Location identifier to contain the rating model. |
| 3 | Label | Rating model label. |
| 4 | InputParameter | Input parameter identifier. |
| 5 | InputUnit | Input unit identifier. |
| 6 | OutputParameter | Output parameter identifier. |
| 7 | OutputUnit | Output unit identifier. |
| 8 | Publish | Optional publish flag. Defaults to `False`. |
| 9 | Description | Optional rating model description. |
| 10 | Comment | Optional rating model comment. |
| 11 | UtcOffset | Optional UTC offset, in +HH:MM or -HH:MM format.<br/><br/>If omitted, the location's UTC offset will be used if available, otherwise the system's local timezone offset will be used. |

```csv
Model, LocA, Primary, Stage, m, Discharge, m^3/s
```

This row creates the `Stage-Discharge.Primary@LocA` rating model.

### `Curve` rows

- For `LinearTable` or `LogarithmicTable` curve tables, one or more `TablePoint` rows should follow the `Curve` row, to define the table.
- For `StandardEquation` curve types, the `Curve` row also includes the `Y = a*(b + X)^c + d` coefficients.

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Curve`. |
| 2 | CurveType | Must be one of `StandardEquation`, `LinearTable`, or `LogarithmicTable`. |
| 3 | Name | Unique name of the curve. |
| 4 | OutputWhenBelow | The output value when the input is below the rating. If omitted, an empty value will be output. |
| 5 | IsNonMonotonic | A non-monotonic rating will be unable to compute shifts or inverse ratings.<br/>`True` or `False`. Defaults to `False` if omitted. |
| 6 | Comment | An optional comment. |
| 7 | A | Standard equation coefficient. |
| 8 | B | Standard equation coefficient. |
| 9 | C | Standard equation coefficient. |
| 10 | D | Standard equation coefficient. |

```csv
Curve, StandardEquation, 001, , , This is my equation, 3.806, -12.23, 1.75, 0
Period, 001, 2017-05-23, 2017-06-24 13:15
```

This row creates a curve named "001" using the equation `Y = 3.806*(-12.23 + X)^1.75 + 0.0`.

### `Period` rows

A rating curve needs to have an applicable period in order to have an effect on the rating-derived output.

It is fine for a curve to have no `Period` rows at all. That curve will exist within the rating, but will never have an effect on the derived output. This is often true for test curves that are under development, or obsolete curves being imported for historical reasons.

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Period`. |
| 2 | Name | Unique name of the curve. |
| 3 | Start | The start time of the period of applicability. |
| 4 | End | The optional end time of the period of applicability. |
| 5 | Comment | An optional comment. |

If none of the curves in a model have any applicable periods, the tool will throw an error, since that is usually an indicator that you've messed up. You can set the `/AllowEmptyModels=true` override option to disable this error check.

While it is fine to define a curve for a model, but not assign with no applicable periods.

When the `End` column is blank or omitted, the end time will be inferred by taking the `Start` value of the next adjacent curve and adding the `/BlendInterval=timeSpan` overlap.

### `TablePoint` rows

At least one `TablePoint` row is required for each `LinearTable` or `LogarithmicTable` curve.

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `TablePoint`. |
| 2 | Input | Input value. |
| 3 | Output | Output value. |

```
Curve, LinearTable, 002, , , A silly table I made
Period, 002, 2017-06-25
TablePoint, 1, 100
TablePoint, 2, 200
```

These rows create a linear table which scales the input by 100.

### `Formula` rows

A `Formula` row can be used as a shortcut to generate a number of `TablePoint` rows for an `LinearTable` or `LogarithmicTable` curve, using an algebraic formula.

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Formula`. |
| 2 | LowInputValue | The lowest input value of the generated table. |
| 3 | HighInputValue | The highest input value of the generated table. |
| 4 | Equation | An algebraic equation, with "x" as an input variable. |
| 5 | MaximumErrorRatio | An optional maximum allowed error ratio. Defaults to 0.0005 (.05% of interpolated output) |
| 6 | MaximumPointCount | An optional maximum table size. Defaults to 2000 points. |
| 7 | RequiredInput1 | Optional required inputs. The converted table will contain this input point and its calculated output. |
| ... | ... | |
| 7+N | RequiredInputN | Optional required input. |

```
Curve, LinearTable, 002, , , A silly table I made
Period, 002, 2017-06-25
Formula, 1, 2, 100 * x
```

These rows create a linear table which scales the input by 100.

The `TablePoints` generated are as few as possible to represent the equation within the `MaximumErrorRatio`.
- Initially, a table of `MaximumPointCount` points is generated, with points equally spaced between `LowInputValue` and `HighInputValue`.
- Each intermediate point is examined, to see if its interpolated value is within the `MaximumErrorRatio` of the formula. If the interpolated value is close enough, the point is removed from the table.
- All `RequiredInputs` and their calculated output values will be retained in the generated table. These required points cannot be optimized away.
- A linear or constant formula will always yield a table of just 2 points (the `LowInputValue` and `HighInputValue` points), assuming no `RequiredInputs` are given.

#### Supported `Formula` math operations

The formula engine:
- Supports all the basic math operators, including `^` for exponentiation, using algebraic order of operations precedence.
- Supports parenthesis
- Supports trigonometric and standard functions like `sin` and `sqrt`.
- Includes predefined constants like `pi`, `e`, and `NaN`.
- Is case insensitive. `PI` and `pi` are the same value.
- Ignores whitespace

See the [JACE Wiki](https://github.com/pieterderycke/Jace/wiki/Basic-Operations) for further details.

Example formulas:

| Formula | Description |
| --- | --- |
| `12.5` | A constant value. |
| `sin(x)` | The sine of the input. |
| `3.806*(-12.23 + X)^1.75 + 0.0` | The [`StandardEquation`](#curve-rows) example from above. |

### `Shift` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Shift`. |
| 2 | Start | The start time of the shift. |
| 3 | End | The optional end time of the shift. |
| 4 | Input1 | The input value of the first shift point. |
| 5 | Shift1 | The shifted value of the first shift point. |
| 6 | Input2 | The optional input value of the second shift point. |
| 7 | Shift2 | The optional shifted value of the second shift point. |
| 8 | Input3 | The optional input value of the third shift point. |
| 9 | Shift3 | The optional shifted value of the third shift point. |
| 10 | Comment | An optional comment. |

```csv
Shift, 2013-02-01, , 0.1, 0.01, 1.5, 0.5
```

This row creates a shift starting Feb 2013, where a stage of 1.5 m is shifted by 0.5 m.

### `Offset` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Offset`. |
| 2 | Offset1 | First offset point. |
| 3 | Offset2 | Optional second offset point. |
| 4 | Breakpoint1 | Optional first breakpoint. |
| 5 | Offset3 | Optional third offset point. |
| 6 | Breakpoint2 | Optional second breakpoint. |

A `LogarithmicTable` row requires an `Offset` row to follow it.

```csv
Offset, 2.9, 4.3, 4.5
```

This row sets an offset for a v-notch weir, with the first offset 2.9 m (the point of zero flow through the notch), and a second offset of 4.3 m with a breakpoint of 4.5 m, for the rectangular portion of the weir.

### `Grade` rows

Grade, Output, GradeCode

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Grade`. |
| 2 | Output | The maximum output-value of the grade to apply. Leave blank to mean "anything above the highest specified value". |
| 3 | GradeCode | The grade code to apply to the output value. |

```csv
Grade, 10, 20
Grade, , -1
```

These rows apply a grade code of 20 to any output value of 20 or below. Any output value above 20 has a grade code of -1 applied.

## Timestamp formats

Date/times in the CSV can:
- Just specify the date in `yyyy-MM-dd` format
- Include an optional time-of-day, in `HH:mm` or `HH:mm:ss` format (midnight is assumed if omitted)
- All timestamps are assumed to be in the resolved timezone of the rating model (see next section)

### Resolving the model's UTC offset

All timestamps in AQTS need to be unambiguous, which requires an exact UTC offset to be known.

When the optional `UtcOffset` value set in column 11 of the `Model` row is set, that value will always be used.

When the `UtcOffset` value is not explicitly set in the CSV, the tool uses the following logic to resolve the UTC offset:
- When the `/Server=hostname` option is set, query the model's location and use its UTC offset.
- When no `/Server` connection is set, use the `/UtcOffset=±HH:MM` value if set, otherwise use the computer's local timezone.

### `Start` is required, `End` is optional

Both rating curve periods and rating curve shifts have `Start` and `End` properties to define their period of applicability.

- The `Start` property is required and defines the first instant in the period of applicability.
- The `End` property is optional, and defines the first instant **after** the period of applicability.

To define a period of the calendar year 2012, use `Start=2012-01-01` and `End=2013-01-01` so that all of 2012 is included.

When the `End` property is omitted, the tool will automatically infer an `End` value by using the `Start` value of the next adjacent period of applicability when possible.

If your periods were entered as:

```
Period, 001, 2012-01-01
Period, 002, 2013-01-01
Period, 001, 2014-01-01
```

Then the tool would resolve them as:

```
Period, 001, 2012-01-01, 2013-01-01
Period, 002, 2013-01-01, 2014-01-01
Period, 001, 2014-01-01
```

Only the last period will be left with an open `End` value.

## Using the command line instead of a CSV file to create a rating model

The tool can also be used to create a rating model using only command line parameters, without a CSV file. This can allow convenient integration into provisioning workflows.

The `rowType=Value1,...,ValueN` syntax can be used to specify an equivalent row of CSV data from the command line.

The following command line creates an empty rating model 'Stage-Discharge.Primary@Dougly'.

```
RatingModelExchange -server=myappserver Model,Dougly,Primary,Stage,m,Discharge,m^3/s
```

## Testing the derived output locally

You can use the `/InputPoints=list-of-numbers` option to run some input values through the rating curves to see what their derived output values and grade codes would be.

### Use `/InputPoints` or `/InputPointsPath` to specify the input values

- `/InputPoints=list-of-numbers` takes a comma-separated list of floating point numbers.
- `/InputPointsPath=file` can also be used to specify a text file of input points, one number per line.

If no input points are specified, then no local derivation tests will be performed.

### Use `/DerivedRatingIdentifier` to restrict the tested rating curves

The `/DerivedRatingIdentifier=identifier` or `/DerivedRatingIdentifier=identifier:curveName` options can be specified to only perform derivation on the named model, or named curve within a model.

If no `/DerivedRatingIdentifier` is set, all rating curves within all rating models will be tested with the same `/InputPoints`.

### Use `/EffectiveTimes` to chose the rating curve in effect at specific times

- The `/EffectiveTimes=list-of-dateTimes` option can be used to select different curves during their periods of applicability.

When no `/EffectiveTimes` values are specified, the current time is used.

### Use `/DerivedOutputPath=path` to save the derived results to a CSV file

If a `/DerivedOutputPath` option is set, the derived results will be written to that file in CSV format

If no `/DerivedOutputPath` option is set, then the derived results will be written to the console.

### Some derived output examples

The following examples use a simple `StandardEquation` to convert from Celcius to Farenheit.

```
Model, Loc, Label, Temp, degC, Temp, degF
Curve, StandardEquation, 1-1, , , , 1.8, 0, 1, 32
Period, 1-1, 2001-01-01
```

This command line will convert the freezing point and boiling point of water into freedom units:

```sh
$ ./RatingModelExchange.exe "Model,Loc,Label,Temp,degC,Temp,degF" "Curve,StandardEquation,1-1,,,,1.8,0,1,32" "Period,1-1,2001-01-01" -inputpoints=0,100
RatingModelExchange (v1.0.0.0) loaded 1 rating models.
Testing all rating models with 2 input points.
Testing 'Temp-Temp.Label@Loc' with 2 input points. (degC) => (degF)
2019-06-20T14:18:03.4774677-07:00: Input=0 Output=32 Grade= Source=Curve 1-1
2019-06-20T14:18:03.4774677-07:00: Input=100 Output=212 Grade= Source=Curve 1-1
```

The next command adds a `/DerivedPointsPath=results.csv` option to save the results to the `results.csv` file.

```sh
$ ./RatingModelExchange.exe "Model,Loc,Label,Temp,degC,Temp,degF" "Curve,StandardEquation,1-1,,,,1.8,0,1,32" "Period,1-1,2001-01-01" -inputpoints=0,100 -derivedpointspath=results.csv
RatingModelExchange (v1.0.0.0) loaded 1 rating models.
Testing all rating models with 2 input points.
Writing derived output to 'results.csv'.

$ cat results.csv
# Testing 'Temp-Temp.Label@Loc' with 2 input points. (degC) => (degF)
EffectiveTime, InputValue, OutputValue, GradeCode, Source
2019-06-20T14:20:06.3205539-07:00, 0, 32, , Curve 1-1
2019-06-20T14:20:06.3205539-07:00, 100, 212, , Curve 1-1
```

## The `/help` screen

```
Purpose: Provision multiple rating models on an AQTS server

Usage: RatingModelExchange [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  -CsvFiles                  The CSV file(s) to parse for rating model definitions.

  ========================== Import ratings to an AQTS system.
  -Server                    The AQTS app server
  -Username                  AQTS username [default: admin]
  -Password                  AQTS credentials [default: admin]
  -UpdateExistingModels      Allow updates to existing models. [default: False]
  -DeleteExistingCurves      Deletes all existing curves before updating existing models. [default: False]
  -DryRun                    Skips updates to any AQTS rating models. The /N shortcut also enables this option. [default: False]

  ========================== Save ratings to disk, for later processing.
  -Export                    Export this existing rating model to CSV and HydroML. '*' will export all models.
  -SaveHydroML               Save the rating model to this HydroML XML file.
  -SaveRoot                  Save the rating model to this TimeSeriesImporter folder.
  -Overwrite                 Overwrite existing files? [default: False]

  ========================== Optional configuration, for non-standard ratings.
  -AllowEmptyModels          Allow models where no curves have applicable periods? [default: False]
  -BlendInterval             Rating blend interval between adjacent curves. [default: No blend]
  -UtcOffset                 Use this UTC offset when the model's offset is not set. [default: Use the location's UTC offset]
  -ParametersJson            Read parameter definitions from this JSON file.

  ========================== Derivation test options, when input points are provided.
  -InputPoints               Adds the comma-separated list of input points for derivation
  -EffectiveTimes            Use the comma-separated list of times for input derivation. [default to current time for all points]
  -InputPointsPath           Read input derivation points from this file, one [timestamp,]value per row.
  -DerivedPointsPath         Write derived points to this file.
  -DerivedRatingIdentifier   Derive points for this specific rating curve in <InputParameter>-<OutputParameter>.<Label>@<Location>[:<CurveName>] format. [default: Derive for all curves if any input points are provided.]
  -DerivationTestLocation    When set, create a temporary time-series in this location to compare derivation results.
  -MaximumDerivationErrors   Maximum allowed derivation errors before stopping comparisons. [default: 20]
  -AdjustToUtc               Enables conversion to UTC for all test effective times. [default: False]
  -Verbose                   Enables detailed calculation information during testing. [default: False]
  -LogRetainedPoints         Set to true to log when points are kept in optimized tables. [default: False]
  -StrictMode                Enables strict CSV validation when reading CSV files. [default: True]
  -ParameterAliases          Adds a parameter alias, in oldId:NewId format

  ========================== Table optimization options, only including points which change the curve shape.
  -OptimizeAllTables         Enables optimization of all tables. [default: False]
  -MaximumErrorRatio         Maximum predictive error which removes a point from the table. [default: 0.0005 (0.05% of interpolated output)]
  -MaximumFormulaPointCount  Number of starting points when creating tables from formulas. [default: 2000]

As an alternative to the /CsvFiles option, you can use 'RowType,Value1,...,ValueN' arguments to define rating model options.

RowType must be one of Model, Curve, Period, TablePoint, Formula, Shift, Offset, Grade.

Ex. RatingModelExchange -server=myserver model,LocA,Primary,Stage,m,Discharge,m^3/s

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.
```