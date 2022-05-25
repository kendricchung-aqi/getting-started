# ProvisioningTool - `DerivedTimeSeries` tasks

- CREATE, UPDATE, and EXPORT operations are supported.
- DELETE operations are not supported for time-series (neither basic, reflected, nor derived). But you can use the [LocationDeleter.exe tool](https://github.com/AquaticInformatics/examples/tree/master/TimeSeries/PublicApis/SdkExamples/LocationDeleter#deleting-time-series) for that.

## CSV file format

The CSV format for creating/updating derived series is similar to the CSV format used by the [[RatingModelExchange]] tool.

- Blank rows are skipped.
- Rows starting with `#` or `//` are treated as comments and are skipped.
- Leading and trailing whitespace is trimmed from each field.
- Quotes are not required for string fields, unless the field contains a comma

9 row types are supported. The first field in each row defines the row type.

- A [`DerivedSeries`](#derivedseries-rows) row creates the outer shell for a derived series.
- All the other rows that follow define specific processing periods, beginning at a `StartingFrom` point in time.
- Each processing period must have a `StartingFrom` timestamp later than all preceeding periods.
- A derived series can have any number of processing periods defined.

| Row type | Description |
| --- | --- |
| [`DerivedSeries`](#derivedseries-rows) | Defines a new derived series. |
| [`NoProcessing`](#noprocessing-rows) | Defines a period of no processing. |
| [`Passthrough`](#passthrough-rows) | Defines a period of passing through the corrected signal from another series. |
| [`Calculation`](#calculation-rows) | Defines a period of using a formula to calculate values from one or more series. |
| [`RatingModel`](#ratingmodel-rows) | Defines a period of rating model derivation from one series through a rating model. |
| [`Statistical`](#statistical-rows) | Defines a period of statistical processing from one series. |
| [`Transformation`](#transformation-rows) | Defines a period of transformation processing. |
| [`FillMissingData`](#fillmissingdata-rows) | Defines a period of filling data gaps in a source series with points from a secondary seris. |
| [`DatumConversion`](#datumconversion-rows) | Defines a period of datum conversion on one series into a specific datum. |

See [Timestamp Formats](#timestamp-formats) for the supported date/time formats

### `DerivedSeries` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `DerivedSeries`. |
| 2 | ParameterId | Parameter ID of the series. This is the ID, not the DisplayName, so 'HG' and not "Stage'. |
| 3 | UnitId | Optional unit Id of the series. |
| 4 | Label | The label for the derived series. |
| 5 | LocationIdentifier | The identifier of the location which owns the derived series. |
| 6 | UtcOffset | Optional UTC offset, in +HH:MM or -HH:MM format. If blank, defaults to the location's UTC offset. |
| 7 | Description | Optional description for the series. |
| 8 | Comment | Optional comment for the series. |
| 9 | Publish | Optional Publish flag, defaults to `false`. |
| 10 | InterpolationType | Optional interpolation type. If set, must be one of: <br/> `InstantaneousValues` <br/> `PrecedingConstant` <br/> `PrecedingTotals` <br/> `InstantaneousTotals` <br/> `DiscreteValues` <br/> `SucceedingConstant` |
| 11 | ComputationIdentifier | Optional computation type. If set, must be one of: <br/> `Min` <br/> `Max` <br/> `Sum` <br/> `Mean` <br/> `Median` <br/> `Selected Value` <br/> `Tidal High` <br/> `Tidal Lower High` <br/> `Tidal Higher Low` <br/> `Tidal Low` <br/> `Decumulated` <br/> `Max At Event Time` <br/> `Total Amount` <br/> |
| 12 | ComputationPeriodIdentifier | Optional computation period. If set, must be one of: <br/> `Annual` <br/> `Monthly` <br/> `Weekly` <br/> `Daily` <br/> `Hourly` <br/> `Minutes` <br/> `Points` <br/> `WaterYear` <br/> |

While there can be many fields in a `DerivedSeries` row, only the first 5 fields are required. The remaining 7 fields are optional and assume reasonable default values.

When the `UnitId` field is not explicitly set:
- Use the unit ID of the first input series of the first processing period.
- If no processing is define, use the parameter's default unit ID.

When the `InterpolationType` field is not explicitly set:
- `PrecedingConstant` if the first processing plan is `Statistical`.
- Else use the interpolation type of the first input time series of the first processing period.
- If no processing is defined, the parameter's default interpolation type will be used.

When the `ComputationIdentifier` field is not explicitly set:
- If any statistical processing is configured, use first statistic's `StatisticType` field value
- Else leave it blank.

When the `ComputationPeriodIdentifier` field is not explicitly set:
- If any statistical processing is configured, use first statistic's `Period` field value
- Else leave it blank.

### `NoProcessing` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `NoProcessing`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |

### `Passthrough` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Passthrough`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| 4 | [InputTimeSeries](#inputtimeseries) | The input time-series |
| 5 | Method | The optional method code. |

### InputTimeSeries

Input time-series can be specified with no location identifier, as `{ParameterId}.{Label}` (eg. `HG.Telemetry`). The location of the derived series will be assumed. This is the most succinct and most common form.

Input time-series can also be specified with an explicit location identifier, , as `{ParameterId}.{Label}@{LocationIdentifier}` (eg. `HG.Telemetry@Loc2`). This form is required when one location's series needs to pull in data from a different location.
### `Calculation` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Calculation`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `RatingModel` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Model`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `Statistical` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Statistical`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `Transformation` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Transformation`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `FillMissingData` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `FillMissingData`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `DatumConversion` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `DatumConversion`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

## Timestamp formats

The `StartingFrom` date/times in each processing period can:
- Be completely blank, to indicate starting from the beginning of record.
- Just specify the date in `yyyy-MM-dd` format
- Include an optional time-of-day, in `HH:mm` or `HH:mm:ss` format (midnight is assumed if omitted)
- All timestamps are assumed to be in the timezone of the series unless an explicit UTC offset is provided.
