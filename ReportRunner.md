Download the [latest ReportRunner.exe here](https://github.com/AquaticInformatics/getting-started/releases/ReportRunner)

# ReportRunner

- It's a single EXE, no installation or unzip operation of 40+ binaries is required.
- Works with AQTS 2017.3-onwards.
- Run a report from the command line.
- Uses standard process exit codes. 0 for success, 1 for errors.
- Supports the [common command line options syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) used by many free Aquatic Informatics tools.
- Logs its output to the standard log location of `%ProgramData%\Aquatic Informatics\AQUARIUS\Logs\ReportRunner.log`
- Works equally well from CMD.EXE, bash, or Powershell.
- Consumes the clipboard JSON from the Report app
- Uses reasonable defaults when it can
- Command-line parameters get merged with anything retrieved from JSON
- Perform time-series identifier => unique ID lookups automatically. You can specify either the unique ID directly or an identifier in "Parameter.Label@Location" syntax.
- Specify report output type (including plot pdf, plot png, pdf, csv, and hopefully xml dataset one day)
- Optionally wait for the report request to complete
- Optionally download the generated file
- Many arguments can be figured out from context. `-TimeSeries="Stage.Label@Location"` and `"Stage.Label@Location"` mean the same thing.

Commands:
- **List** => lists all installed reports
- **CreateTemplate** => saves a JSON of a report's parameter list
- **Run** => queues the report request. Optionally wait until complete

## Compatibility with previous `ReportRunner.exe` tools

Versions of `ReportRunner.exe` before 2018.3 were much more limited in functionality.

- Previous version were distributed as a `ReportRunner.zip` archive, containing 40+ files. This archive needed to be unzipped and deployed *somewhere*. The new version is single EXE, which can be run from any folder.
- Command lines which worked with older versions of `ReportRunner` continue to work, so the new ReportRunner is a drop-in replacement.

## Basic help. Try the `-help` or `/?` option

```
Run AQTS report requests from the command-line.

usage: ReportRunner [-option=value] [@optionsFile] [command] [timeSeriesIdentifier] [template.json] [output.pdf|csv] [relativeTime]

Supported -option=value settings (/option=value works too):

======================== Server credentials
-Server                  AQTS server name [default: localhost]
-Username                AQTS user name [default: admin]
-Password                AQTS password [default: admin]

======================== Report parameters, which override any values loaded from JSON
-ReportType              The report output type.
-ReportName              The name of AQTS report
-Locale                  The locale of the generated report [default: en]
-RelativeTimeRange       Sets the relative time range for the report
-UtcOffset               Set the UTC Offset used in report requests. Defaults to the UTC offset of the first time-series.
-IsTransient             Set to false to keep the report permanently. Uses the JSON template value, if present, otherwise defaults to true.
-TimeSeries              Sets a time-series identifier report parameter
-LocationInput           Sets a location input report parameter
-Tags                    Sets a report tag. See below for details.

======================== Useful side-effects
-Command                 The command to perform. One of List, CreateTemplate, Run. [default: Run]
-JsonTemplate            The report template saved as JSON
-SaveOutputFile          Save the generated report to this file
-WaitForComplete         When True, wait for the generated report to complete [default: False]
-RemoveDuplicateReports  When True, deletes any generated reports with the same ReportTitle. [default: True for permanent reports]

======================== Compatibility options. Prefer using other options describe above.
-ParametersFilePath      Compatibility alias for /JsonTemplate= option
-Login                   Compatibility alias for /Username= and /Password= options
-LogFileName             Compatibility alias (no longer has any effect)

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.

Relative Time Range Syntax: <type>[/<value>][/<unit>][/<value2>]

  <type>   - One of LatestComplete, MostRecent, WaterYear, Year, Month, Day, DateRange, EntireRecord.
  <value>  - Required for all types except EntireRecord. Usually an integer, but could be an ISO-8601 timestamp.
  <unit>   - Required for all types except EntireRecord and DateRange. One of None, WaterYears, Years, Months, Days.
  <value2> - Optional secondary value, an ISO-8601 timestamp. Only used for DateRange type.

Ex. /RelativeTimeRange=MostRecent/2/Days - The most recent 2 days
    /RelativeTimeRange=DateRange/2017-04-01T00:00:00Z/2017-04-02T00:00:00Z - All of April 1st, 2017
    /RelativeTimeRange=DateRange/2017-05-01/2017-05-31 - All of May 2017

UTC Offsets can be +HH:MM, -HH:MM, or ISO 8601 durations (Ex. PT2H or -PT3H30M).

Tag Syntax: The /Tags= option can be specified multiple times.

  /Tags=TagName           - Specify a simple tag by name.
  /Tags=PicklistTag:Value - Specify a picklist tag by name, colon, and value.
  /Tags=                  - An empty value clears all exist tags from the request.

Unknown /name=value options not listed above will be treated as report parameters.
Command-line parameters will override JSON template parameters of the same name.
```

# Scheduling a report to be run periodically.

This is the main use case for `ReportRunner.exe`.

AQTS 2019.3 ships with `ReportRunner.exe` installed on the server at this default path: 
`%ProgramFiles%\Aquatic Informatics\AQUARIUS Server\ReportPluginPackages\ReportRunner.exe`.

You can also download `ReportRunner.exe` from [AI Source](https://github.com/AquaticInformatics/getting-started/releases/tag/ReportRunner) and run it from any Windows computer on your network.

Scheduling a report to be generated periodically involves three steps:
- Create a JSON report template with all the settings required to create the report (outlined below)
- Having `ReportRunner.exe` installed on a system somewhere on your network (it can be run from the app server, or from another Windows computer on your network)
- Using a standard scheduling tool (like Windows Task Scheduler) to run `ReportRunner.exe` with the JSON template and any other command line options.

Successful scheduling of report requests will require an understanding of the `ReportRunner.exe` command-line options.
`ReportRunner.exe` uses the [common command line options syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) used by many free Aquatic Informatics tools.
Integrators are strongly encouraged to use the `@filename.ext` approach to store all the command-line options in a text file, to simplify the command-line signature and to document why special options are used.

## Example - DailyMeanLast3Months

This example makes a few assumptions:
- Runs on the AQTS app server, since `ReportRunner.exe` is included with the server installation from AQTS 2019.3 onwards.
- A new directory has been created at `C:\Reports` to contain both JSON report templates and `@options.txt` command-line text files.
- A JSON report template for the daily mean of the last 3 months has been saved to `C:\Reports\DailyMeanLast3Months.json`
- A command line options text file to invoke the report has been saved to `C:\Reports\DailyMeanLast3Months.txt`

We'll schedule this report to run every morning at 7 AM local time, using the Windows **Task Scheduler**:
- Click **Start -> All Programs -> Accessories -> System Tools -> Task Scheduler**
- From the **Action** menu on the right, Select **Create Basic Task**. Give the task a name, select a **Daily** trigger, and enter 07:00 AM as the desired start time.
- For **Action**, select **Start a Program**.
- For **Program/scipt**, click the Browse button, and select the location of `ReportRunner.exe'.
- For **Add arguments**, enter `@DailyMeanLast3Months.txt` (there should be no space between the `@` sign and the filename)
- For **Start in:**, enter `C:\Reports` as the default working directory.
- Click **Next**, then **Finish**.

![Windows Task Scheduler](https://user-images.githubusercontent.com/54773/137524607-57c04106-a042-45da-979f-bd28d7650236.png)

The `C:\Reports\DailyMeanLast3Months.txt` file can be as simple as these 7 lines:

```
# Specify the AQTS credentials used to run the report
/Server=localhost
/Username=myaccount
/Password=mysekret

# The JSON report template is in the same folder as this text file.
DailyMeanLast3Months.json
```

Note that each time this task is run, a report request will be queued asynchronously, but the task will exit before the report is actually generated (usually within a few seconds).

# Saving a JSON report template from the Report app

Any report which can be run from the Report browser app can also be run from the command line using `ReportRunner.exe`.

The basic workflow to create the report template is:
- Log into Springboard
- Launch the Report app in a new browser tab
- Tweak the report request parameters (title, description, date ranges, output format, etc) until the generated report matches your needs.
- Once you are satisfied with the resulting report, then create a JSON template containing all the current report request settings.

Saving the JSON template is currently an awkward set of 7 steps. Sorry about that.

It *should* be a simple button click that immediately downloads a JSON file to your browser, but right now it is currently a multi-step sequence documented below. Hopefully this clunky experience will be improved in a future AQTS release.

### Step 1 - Click the "View all settings used by this report" link to open a new browser tab

The new page will show all of the report settings in a structured text display.

**Note:** This structured text looks suspiciously like JSON, but it actually isn't valid JSON. If you try to select this text using your mouse, it will contain extra "+" and "-" characters which aren't valid JSON. So instead, proceed to step 2.

![Report Parameters Json](https://user-images.githubusercontent.com/54773/137524660-5272daca-e8d6-4e45-bee6-5e403552a823.png)

### Step 2 - Click the "Copy to Clipboard" button to reveal a 3-line text window

The revealed window will contain all the settings as valid JSON, and the entire JSON text payload will already be selected.

![Copy Json To Clipboard](https://user-images.githubusercontent.com/54773/137524682-68906e32-635d-4adc-a62e-3095bf2ce212.png)

### Step 3 - Type Ctrl-C to copy the selected text to the clipboard

Typing Ctrl-C will copy the JSON payload to the clipboard

### Step 4 - Open a blank text document in a text editor

We need to paste the clipboard contents into a simple text document.

Use an editor that works on plain text files, like Notepad, [Notepad++](https://notepad-plus-plus.org/), or [Visual Studio Code](https://code.visualstudio.com/).

Don't use a rich text editor like Microsoft Word or WordPad, since those editors will add extra characters that aren't valid for JSON documents.

### Step 5 - Paste the JSON settings into the blank document

Ctrl-V will paste the JSON payload from step 3 into the blank text document.

### Step 6 - Optional: Reformat the single line JSON into multi-line JSON

The JSON payload is formatted as a single, very long line.

This is totally fine, it is valid JSON. You could just save the JSON as-is.

But the single line format it is very hard for humans to read and reason about.

![Notepad Single Line Json](https://user-images.githubusercontent.com/54773/137525078-6909bdc2-d9a3-4d8e-81ee-f2d40c38fa29.png)

Many text editors like Notepad++ or Visual Studio Code will have options to quickly reformat JSON documents.

- Notepad++ - `Ctrl` `Shift` `Alt` `M`
- VSCode - `Shift` `Alt` `F`

![Notepad Multi Line Json](https://user-images.githubusercontent.com/54773/137525101-4fd4e2c0-1be6-4d93-94d9-6eab8b3d8244.png)

### Step 7 - Save the file with a ".json" file extension

Save the text file with a ".json" file extension.

This example was a Computation Chain report for the last 3 months of data, so a good filename might be `ComputationChainLast3Months.json`.

Using a ".json" file extension will allow ReportRunner to assume that the file is a JSON report template.

# Get inspired by these `ReportRunner` examples:


## 1 - Run a report using a JSON report template saved from the Report app

This is like previous versions of `ReportRunner.exe`. It simply queues a report request from a JSON file stored locally. It does not wait for the report to actually be generated.

```bat
C:\> ReportRunner.exe /server=doug-vm2012r2 WindRose.json

15:10:10.831 INFO  - Connected to http://doug-vm2012r2/AQUARIUS/Publish/v2 (2018.1.95.0)
15:10:10.849 INFO  - Loading JSON report template 'WindRose.json' ...
15:10:11.064 INFO  - Queued report job 181356 without waiting for report to complete.
```

Note that since the template filename ends with ".json", ReportRunner figured out that you meant `/JsonTemplate=WindRose.json` and saved you a bit of typing.

## 2 - Save the generated report

Now add a PDF filename to the command line, and the tool will wait for the report to be generated and will download the PDF to the given filename.

```bat
C:\> ReportRunner.exe /server=doug-vm2012r2 WindRose.json rose.pdf

15:16:05.614 INFO  - Connected to http://doug-vm2012r2/AQUARIUS/Publish/v2 (2018.1.95.0)
15:16:05.632 INFO  - Loading JSON report template 'WindRose.json' ...
15:16:05.819 INFO  - Waiting for report job Id=181357 ...
15:16:18.962 INFO  - Report job 181357 completed in 13.1 seconds.
15:16:19.373 INFO  - Saved 476.5 KB to 'rose.pdf'.
```

If the report supports CSV output, you can save a CSV file too.

## 3 - Use a JSON template with different time-series

This is one of the more powerful use cases for ReportRunner.

- Use the Report app to tweak all the report parameters until your report output is to your satisfaction.
- Click the "View all settings used by this report" link and download the settings as a JSON template.
- Run ReportRunner with that template, and just override a few parameters (like the time-series or the date range) on the command line.

```sh
$ ./ReportRunner.exe -server=doug-vm2012r2 WindRose.json rose.pdf "Wind Dir.Work@01372058" "Wind Vel.Work@01372058" -ReportTitle="Wind rose for 01372058"

14:39:00.486 INFO  - Connected to http://doug-vm2012r2/AQUARIUS/Publish/v2 (2018.2.100.0)
14:39:00.505 INFO  - Loading JSON report template 'WindRose.json' ...
14:39:00.760 INFO  - Waiting for report job Id=182811 ...
14:39:09.407 INFO  - Report job 182811 completed in 8.6 seconds.
14:39:09.501 INFO  - Saved 477.54 KB to 'rose.pdf'.
14:39:09.520 INFO  - Deleted transient report
```

In this example, ReportRunner saves you a few more keystrokes, by recognizing the `<Parameter>.<Label>@<LocationIdentifier>` syntax in context, so you don't need to provide the `/TimeSeries=` prefix.

## 4 - Show the reports installed on the server

```sh
$ ./ReportRunner.exe -server=doug-vm2012r2 list

15:02:49.469 INFO  - Connected to http://doug-vm2012r2/AQUARIUS/Publish/v2 (2018.1.95.0)
15:02:49.547 INFO  - There are 7 reports installed.
15:02:49.548 INFO  - ReportName='Change List Report' OutputTypes=Pdf
15:02:49.549 INFO  - ReportName='Computation Chain' OutputTypes=Pdf,Csv,PlotPdf,PlotPng
15:02:49.549 INFO  - ReportName='Daily Mean by Year' OutputTypes=Pdf
15:02:49.551 INFO  - ReportName='Event Analysis' OutputTypes=Pdf,Csv
15:02:49.552 INFO  - ReportName='Intensity Analysis' OutputTypes=Pdf,Csv
15:02:49.552 INFO  - ReportName='Profile Plot' OutputTypes=Pdf
15:02:49.553 INFO  - ReportName='Wind Rose' OutputTypes=Pdf
```

ReportRunner recognizes the command keywords `List`, `CreateTemplate`, and `Run` without needing the `/Command=` prefix, to save you a few keystrokes.

## 5 - Create a JSON template from the command line

```sh
$ ./ReportRunner.exe -server=doug-vm2012r2 -ReportName="Change List Report" CreateTemplate -JsonTemplate=ChangeList.json

15:20:25.356 INFO  - Connected to http://doug-vm2012r2/AQUARIUS/Publish/v2 (2018.1.95.0)
15:20:25.507 INFO  - Loaded report definition for 'Change List Report'
15:20:25.509 INFO  - Overriding template IsTransient=False with True
15:20:25.510 INFO  - Overriding template RequestingUserLocale='' with 'en'
15:20:25.510 INFO  - Overriding template OutputType=Unknown with Pdf
15:20:25.513 INFO  - Creating 'ChangeList.json'.
```

The JSON template generated will contain all the default parameter values.

## 6 - Some more examples 

```cmd
ReportRunner.exe -ReportName="Daily Statistic by Year" -RelativeTimeRange="MostRecent/2/Years" -Statistic="Sum" -DataCoverageThresholdPercent=80 -ReportTitle="Daily Statistic by Year" -Description="Daily Sum" -TimeSeries=5fbf488c53314175bd7aee442810e591

ReportRunner.exe -ReportName="Double Mass Plot" -RelativeTimeRange="MostRecent/20/Years" -PlotType=DoubleMass -IntervalCount="1" -IntervalUnit=Month -ReportTitle="Double Mass" -Description="Most Recent 20 years" 5fbf488c53314175bd7aee442810e591 29ad1f3d40b844d9826edcc640a2afee
```