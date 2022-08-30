# ProvisioningTool - `GlobalSetting` tasks

- EXPORT, CREATE, UPDATE, and DELETE operations are supported.

## Format of the `GlobalSettings.csv` file

| Column Header | Example value| Description |
|---|---|---|
| Group| `Administration` | The setting group. |
| Key| `SystemMessage` | The setting key within the group. |
| Value| `Scheduled upgrade planned for Friday 5PM local time. Save your work!` | The setting value as text. Setting text can be any size, can be multiline. <br/><br/>If the value starts with a `@` and the remainder of the value resolves to an existing file, the contents of that file will be used as the setting value. See the [File-based settings](#file-based-settings) section below for more details. |
| Description | `System wide message to be displated to all users.` | An optional description. |

The Group and Key columns are required for CREATE, UPDATE, and DELETE operations.

## File-based settings

Some configurable settings are actually entire text files, spanning multiple lines. Often these settings are JSON or XML documents, used to configure certain AQTS components (often report or field data plugins).

These documents can be stored directly in the CSV `Value` column, but the will need to follow the CSV-escaping rules of:
- Surrounding the entire value in double quotes `"`.
- Double-escaping any double-quote character `"` with two consecutive double qutoes `""`.

This is supported, but it is confusing to read, since double-quotes are common in both JSON and XML, and double-double-quotes are just unwieldy.

### Eg. A JSON document for a field data plugin

Let's assume that you are configuring the [SxS Pro field data plugin](https://github.com/AquaticInformatics/sxs-pro-field-data-plugin#sxs-pro-field-data-plugin).

The JSON configuration for the expected time and date formats parsed by the plugin is this 4-line JSON document.

```json
{
  "DateFormats": [ "M/d/yyyy", "M-d-yyyy", "yyyy/M/d", "yyyy-M-d" ],
  "TimeFormats": [ "h:m:s tt", "h:m tt", "H:m:s", "H:m" ]
}
```

Representing that setting in a CSV row, with all the double-double-quotes, can be done like this:

```csv
Group,Key,Value,Description
FieldDataPluginConfig-SxSPro,Config,"{
  ""DateFormats"": [ ""M/d/yyyy"", ""M-d-yyyy"", ""yyyy/M/d"", ""yyyy-M-d"" ],
  ""TimeFormats"": [ ""h:m:s tt"", ""h:m tt"", ""H:m:s"", ""H:m"" ]
}",Our custom config for the plugin timestamps.
```

That is doable, but quickly becomes unweildy.

### `Value` columns starting with `@` are treated like file paths

The **GlobalSetting** CSV reader supports a special syntax when the `Value` column:
- Is a single line.
- Starts with an `@` character.
- The remainder of the column is a file path which resolves to an existing file on the computer running the ProvisingTool executable.

When all three conditions are met, the contents of the existing file is used as setting value.

That would allow the previous SxS Pro JSON configuration setting to be stored at `C:\Users\SusanSmith\OurSpecialTimestamps.json` as:

```json
{
  "DateFormats": [ "M/d/yyyy", "M-d-yyyy", "yyyy/M/d", "yyyy-M-d" ],
  "TimeFormats": [ "h:m:s tt", "h:m tt", "H:m:s", "H:m" ]
}
```

And referenced from a CSV that looks like this:

```csv
Group,Key,Value,Description
FieldDataPluginConfig-SxSPro,Config,@C:\Users\SusanSmith\OurSpecialTimestamps.json,Our custom config for the plugin timestamps.
```

This makes importing arbitrary XML or JSON configuration settings much more robust.

### File-based settings are automatically exported as separate files

When the EXPORT operation is performed, any XML or JSON documents are automatically exported as separate files in the same folder as the exported CSV file.
