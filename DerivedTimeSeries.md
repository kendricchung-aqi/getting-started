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
| 2 | Location | Location identifier to contain the rating model. |

### `NoProcessing` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `NoProcessing`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

### `Passthrough` rows

| # | Field name | Description |
| --- | --- | --- |
| 1 | RowType | Must be `Passthrough`. |
| 2 | StartingFrom | Optional [starting time](#timestamp-formats) of the processing period. |
| 3 | Description | Optional description of the processing period. |
| x | X | |

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

The `StartingFrom` date/times in the CSV can:
- Be completely blank, to indicate starting from the beginning of record.
- Just specify the date in `yyyy-MM-dd` format
- Include an optional time-of-day, in `HH:mm` or `HH:mm:ss` format (midnight is assumed if omitted)
- All timestamps are assumed to be in the timezone of the series unless an explicit UTC offset is provided.
