Download the [latest AqsAttachmentUploader.exe here](https://github.com/AquaticInformatics/getting-started/releases/AqsAttachmentUploader)

# AqsAttachmentUploader

This console utility can be used to upload many attachments to an AQUARIUS Samples system.

- A single EXE .NET console utility for uploading many attachments at once.
- Files to upload are organized by folders named as location identifiers.
- ZIP archives containing files to upload can also be used.
- Filenames containing date patterns are assumed to be field visit attachments for the same date
- Otherwise files will be uploaded as location attachments.
- This tool supports the flexible [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) for when the command line gets a bit daunting.

Some convenience features are available:
- An `AqsAttachmentUploader.log` file is created as a record of the activities performed.
- Defaults to a "dry-run" option, so you can see which files would be uploaded as location or field visit attachments.
- If an attachment of the same name and file size exists at that location or field-visit, it is not re-uploaded. This allows a failed upload attempt to be quickly resumed after correcting any errors.
- Configurable location aliases are supported.
- Large video attachments can automatically be converted to MP4 video files before uploading. This tends to save disk space and yields a better browser playback experience.

## Using the Dry-run option

The `/DryRun=true` option (or its `/N` shortcut) can be used to see how the `AqsAttachmentUploader` will interpret all the files being processed for uploading.

This is the default behaviour of the tool, so that you don't start accidentally uploading files to unexpected landing spots.

This option is useful when debugging the layout of files or ZIP archives being uploaded.
You can inspect the `AqsAttachmentUploader.log` for `WARN` messages, and resolve any complaints before trying the uploads for real.

|`WARN` message pattern|Resolution|
|---|---|
|Skipping unknown location identifier '*path*'<br/>Skipping file '*path*' with no location folder context. | Add a [location alias](#location-identifier-aliases). |
|No field visit time extracted from '*path*'. | Add a [`/DateTimeFormat`](#specifying-field-visit-datetime-patterns-in-filenames) option. |
|Can't find existing visit at *date* for '*path*' | Upload the missing visit and try again. |
|Skipping big file '*path*' | None. |

## Turn off Dry-run mode once you are ready to start uploading files.

Once you are happy with how the tool will categorize your attachments, you can start uploading attachments to AQUARIUS Samples.

Use the `/DryRun=false` option (or its `/Y` shortcut) to disable the dry-run and start uploading attachments.

## What if I accidentally upload files I shouldn't have?

Well, this is embarassing, isn't it?

You will need to use the Samples UI to delete each individual file that you accidentally uploaded.

## How fast will files be uploaded?

The speed at which attachments are uploaded to AQS will vary based on client, server, and network latency.

But internal testing has seen the tool upload more roughly 2000 attachments per hour.

## How should the files be organized for speedy uploading?

The `/Root=path` option specifies the root path to inspect for attachment candidates.

- Under the root path, folders should be named with AQS location identifiers (case-insensitive).
- Files contained by a location folder will be attached to that location.
- Field visit files can be contained in a known subfolder and use a filename containing the date of a matching field visit.
- When no matching field visits are found, the files will just be uploaded as location attachments instead.

### Example Layout 1 - Just files

Consider this layout of attachments to import:

```
Root\
  Loc1\
    SiteGuide.pdf
    MainPhoto.jpg
    Visits\
      2019-08-11 - Gauge house photo.jpg
      2016-04-01 - CrazyJulyFloodMeasurement.ft
  Location 2\
    EquipmentInventory.xls
    SiteGuide.pdf
```

This layout will attempt to upload files to the location named "Loc1" and to the "Location 2" location.

If Loc1 has an August 11th visit in 2019, then the photo will be attached to the visit.
If Loc1 has an April 1st visit in 2016, then the FlowTracker2 measurement will be attached to the visit.

### Using ZIP archives to migrate attachments

- The `/InspectZipArchives` option defaults to true.
- Any "*.ZIP" files encountered will be inspected and processed as if the ZIP file was already extracted locally.
- If the ZIP archive is within the location folder, all files within the ZIP archive are assumed to belong to the location.
- If the ZIP archive is at the root folder, then the first folder within the ZIP archive is assumed to be a location identifier.
- If you need to actually upload the ZIP archive, instead of each of its files, you will need to specify the `/InspectZipArchives=false` option.

These ZIP-processing rules allow for very flexible collection of attachments to migrate (especially when files are coming from different legacy systems).

Feel free to mix and match any combination of ZIP or non-ZIP files as needed.

With these rules in place, let's see how Example 1 might be stored

### Example Layout 2 - ZIPs for some locations

Let's pack everything for Loc1 in a ZIP archive. The new layout might look like this:

```
Root\
  Loc1\
    AllLoc1Files.zip
  Location 2\
    EquipmentInventory.xls
    SiteGuide.pdf
```

`AllLoc1Files.zip` might look like this:
```
SiteGuide.pdf
MainPhoto.jpg
Visits/
  2019-08-11 - Gauge house photo.jpg
  2016-04-01 - CrazyJulyFloodMeasurement.ft
```

The result of importing Example 1 vs. Example 2 is the same.

### Example Layout 3 - Everything all zipped up

Now let's pack everything into a single ZIP archive. The new layout might look like this:

```
Root\
  AllTheFiles.zip
```

Yup, just a single (big) ZIP file in the root folder. `AllTheFiles.zip` should look like this:
```
Loc1/
  SiteGuide.pdf
  MainPhoto.jpg
  Visits/
    2019-08-11 - Gauge house photo.jpg
    2016-04-01 - CrazyJulyFloodMeasurement.ft
Location 2/
  EquipmentInventory.xls
  SiteGuide.pdf
```

The result of importing Example 1 vs. Example 2 vs. Example 3 is the same.

## Location identifier aliases

By default, the `/Root=path` path folder is expected to contain folders named with AQS location identifiers.

But locations are often renamed during migration, and legacy attachments are often extracted into ZIP archives with their legacy identifier values.

Use the `-LocationAlias=alias:identifier` option to create an alias from a legacy system to a current AQS location identifier.

The `-LocationAlias=` prefix is optional.
You can just specify "alias:identifier" as a positional command line option, and the tool will recognize it as an alias.

Using the [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) is a handy way to specify multiple aliases.

If the `C:\FilesToUpload\AllTheFiles.zip` archive from Example 3 was produced by some legacy system, and the true AQS locations where 1111 and 2222, then this command line would work:

```cmd
C:\> AqsAttachmentUploader.exe -server=myorg.aqsamples.com -Root=C:\FilesToUpload @AllOptions.txt
```

Where `AllOptions.txt` contained these 3 lines (note the use of positional arguments):
```
# Add all the location aliases here, in alias:identifier form. Whitespace around the colon is OK.
Loc1 : 1111
Location 2 : 2222
```

Then the files would be uploaded to the 1111 and 2222 locations, instead of seeing `WARN` statements that 'Loc1' and 'Location 2" are unknown locations.

## Converting video files to MP4

Many common video file formats are quite large. (AVI files and MOV files from phones are generally large and uncompressed). AQSamples has a maximum file upload size of 20 MB per file. It is very easy for uncompressed video to exceed this limit.

The uploader tool can detect these video formats (and others) and use the `ConvertToMP4.exe` tool to convert these larger files into compressed MP4 video files, which tend to be 4x-10x smaller for the same resolution, and play back nicely in all modern browsers.

Set the `/ConvertToMP4Path=somePath` option to point to the `ConvertToMP4.exe` tool, and the `AqsAttachmentUploader` will automatically convert every video file to an MP4, and then upload the MP4 instead of the larger AVI/MOV. You will avoid most file upload size errors and not notice any drop in video/audio quality.

If no `/ConvertToMP4Path` option is set, the same directory as `AqsAttachmentUploader.exe` is searched.

If the conversion tool is not found, no video conversion will be attempted.

```
...
06881500: Location attachment '06881500/HW Photos/05-08-2015/018.JPG' (3.7 MB)
Converting '06881500/HW Photos/05-08-2015/020.AVI' to MP4 ...
ConvertToMP4 (v1.0.499): Converting 'C:\Users\Doug.Schmidt\AppData\Local\Temp\020.AVI' to 'C:\Users\Doug.Schmidt\AppData\Local\Temp\020.mp4' ...
Converted 'C:\Users\Doug.Schmidt\AppData\Local\Temp\020.AVI' (165.3 MB) to 'C:\Users\Doug.Schmidt\AppData\Local\Temp\020.mp4 (23.3 MB), 14% of original, in 12 seconds, 494 milliseconds.
06881500: Location attachment '06881500/HW Photos/05-08-2015/020.mp4' (23.3 MB)
...
```
## Detecting field visit attachments

The `AqsAttachmentUploader` tool tries to detect field visit attachments using four rules and three command line options.

- The `/FieldVisitSubFolders=name1,...,nameN` list specifies folders which contain field visit files.
- The `/DateTimeFormats=pattern1,...,patternN` list specifies patterns to extract timestamps from filenames.
- The `/AttachmentInfo=path` option can provide [explicit mappings of files](#the-attachmentinfopath-option-controls-where-attachments-are-uploaded) to locations/visits.

The two list options have some common behaviour:
- Each list has some reasonable defaults.
- To reset the list, just specify an empty value after the equals sign. (so `/DateTimeFormats=` will discard the default datetime patterns)
- Each option can be specified multiple times, and any values are added to the current list.
- Using an `@options.txt` file usually helps keep the command line sane.

The 4 rules for matching field visit attachments are:

### Rule 1) Does a matching Filename column exist in the `/AttachmentInfo` file?

When the [`/AttachmentInfo=path` option](#the-attachmentinfopath-option-controls-where-attachments-are-uploaded) is specified, and a CSV row has a **Filename** column matching the *relative source file path*, then rules 2 through 4 will not apply.

The *relative source file path* is the path to the discovered attachment (including the path within a containing ZIP file), relative to the `-Root=path` root folder.

The correct CSV **Filename** column value will depend on the layout of your source attachment files.

- `Loc1\SiteGuide.pdf` will match the first location's site guide from [Example 1](#example-layout-1-just-files).
- `Loc1\AllLoc1Files.zip\SiteGuide.pdf` will match the first location's site guide from [Example 2](#example-layout-2-zips-for-some-locations).
- `AllTheFiles.zip\Loc1\SiteGuide.pdf` will match the first location's site guide from [Example 3](#example-layout-3-everything-all-zipped-up).

When no CSV rows match the *relative source file path*, or if no `/AttachmentInfo=path` option is specified, then rules 2 through 4 must be used to determine where an attachment belongs.

### Rule 2) Does the file path match a `/FieldVisitSubFolders` name?

- Look at the file path to see if contains a folder exactly matching one of the expected names (which defaults to a single name of "Visits").
- More than one folder name can be specified by using a comma-separated list. `/FieldVisitSubFolders=Visits,Measurements` names.
- Folder name matching is case-insensitive.

### Rule 3) Does the filename match a `/DateTimeFormats` pattern?

- Look at the filename (but not the path) and see if one of the `/DateTimeFormats` patterns is matched.
- If no datetime pattern is matched, a `WARN` message is logged.

### Rule 4) Does the datetime extracted from the filename match an existing field visit?

- If a timestamp is extracted, and a visit exists at the location, then the file is uploaded as a visit attachment.
- If a timestamp is extracted but no matching visit is found, a `WARN` message is logged.

### The `/UploadUnknownVisitAttachments=true` option can force unmatched visit attachments to their location

When the `/UploadUnknownVisitAttachments` option is set, any files failing rule 3 or rule 4 will still be uploaded as a location attachment.

Usually you want to leave this option off, so that the tool can be re-run later, once the missing visits are uploaded.

### Specifying field visit datetime patterns in filenames

The `/DateTimeFormats` pattern strings are [.NET custom date/time format strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings).

These format strings can be rather fussy to deal with, so take care to consider some of the common edge cases:
- Format strings are case-sensitive. Common mistakes are made for month-vs-minute and 24-hour-vs-12-hour patterns.
- Uppercase ['M'](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings#M_Specifier) matches month digits, between 1 and 12.
- Lowercase ['m'](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings#mSpecifier) matches minute digits, between 0 and 59.
- Uppercase ['H'](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings#H_Specifier) matches 24-hour hour digits, between 0 and 23.
- **Don't use lowercase ['h'](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings#hSpecifier)**, which only matches 12-hour hour digits, between 1 and 12, and require a ['t' or 'tt'](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings#tSpecifier) pattern to distinguish AM from PM.

Patterns are matched in the order they are specified. If you are getting mismatched field visit attachments, try changing the order of the `/DateTimeFormat` options.

## The `/AttachmentInfo=path` option controls where attachments are uploaded

#the-attachmentinfopath-option-controls-where-attachments-are-uploaded

The `-AttachmentInfo=csvpath` option can be used to:
- Provide an explicit mapping of attachments to a target location or target field visit.
- Provide a comment for the attachment.
- Override the attachment filename stored in AQUARIUS Samples.

This CSV file can be useful to control which attachments get uploaded to which locations and visits, when no implicit mapping can be inferred from the layout of the source folder or ZIP archive.

The CSV has the following shape:

```csv
"LOCATION","FILENAME","VISITDATE","UPLOADEDFILENAME","COMMENTS"
"000013","WQ_DOCS.zip/WQ_DOCS/WQ_0000406.pdf",11-JUL-2012,"AAA7521_JUL_2012.pdf","Monthly report"
"000013","WQ_DOCS.zip/WQ_DOCS/WQ_0000405.pdf",04-OCT-2012,"AAA7521_OCT2012_WQ.pdf,
...
```

- The header columns are case-insensitive.
- The header row must exist, but only the **Location**, **Filename**, and **Comments** columns are required.
- The **UploadedFilename** and **VisitDate** columns are optional.
- The order of header columns does not matter.
- Leading and trailing whitespace is ignored around each column value
- Columns can optionally be enclosed in double-quotes (only required required when the column value includes a comma)

| Column | Required? | Description |
|---|---|---|
| **Location** | Y | The AQUARIUS Samples sampling location ID, or [location alias](#location-identifier-aliases). |
| **Filename** | Y | The source path of the file, relative to the **-Root**=*folder* path. Forward-slashes `/` and backward-slashes `\` within the path are treated identically.<br/><br/>When the source file is within a ZIP archive, the source path is the combination of the relative path to the ZIP plus the path within the ZIP archive.<br/><br/>`somefile.pdf` - Will match root\somefile.pdf<br/>`archive.zip/somefolder/somefile.pdf` - Will match the "somefolder\somefile.pdf within the root\archive.zip|
| **Comments** | Y | When not empty, the text will be added as a comment to the attached file. |
| **UploadedFilename** | N | When not empty, the attachment will be uploaded with this filename, instead of the source filename. |
| **VisitDate** | N | When not empty, this column specifies the date of the field visit to receive the attachment. Most date formats are accepted, including:<br/><br/>- yyyy-MM-dd `2019-04-15` <br/>- dd/MMM/yy `15/APR/19` |

## Exporting existing attachments

The tool can also export existing attachments from AQS into a folder or ZIP file, which could be used to re-import the attachments into another AQS system with the same locations and visits.

- `-Export=folder` will export the attachments to a folder, with a location subfolder for each exported location.
- `-Export=file.zip` will export the attachments to a ZIP file, to conserve storage space.
- Use the `-ExportLocations=loc1,loc2` option to specify a subset of location identifiers to export, otherwise all locations will be exported.

## Help screen

```
Upload attachments to an AQUARIUS Samples system.

Usage: AqsAttachmentUploader [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  =============================== Upload attachments to an AQUARIUS Samples system.
  -Server                         The AQUARIUS Samples server
  -ApiToken                       AQUARIUS Samples API token

  =============================== Attachment processing options.
  -Root                           The root folder containing attachments organized by location identifier folders.
  -InspectZipArchives             Inspect ZIP archives for attachments [default: True]
  -DryRun                         Don't upload the attachments. Use /N or /Y as a shortcut for /DryRun=True or /DryRun=False [default: True]
  -FieldVisitSubFolders           Folder names indicating field visit attachments [default: Visits]
  -DateTimeFormats                Datetime formats to match in filenames. [default: yyyy-MM-dd]
  -UploadUnknownVisitAttachments  Upload unknown visit attachments as location attachments instead. [default: False]
  -IgnoreFiles                    Filenames to ignore [default: .ppinfocache, Thumbs.db, Desktop.ini, .DS_Store]
  -UploadLimitMB                  Upload file size limit in megabytes [default: 20]
  -ConvertToMP4Path               Path to the ConvertToMP4.exe utility, used to convert videos to MP4 format before upload.
  -LocationAlias                  Add location aliases in LocationAlias=alias:locationIdentifier form

  =============================== Attachment mapping options.
  -AttachmentInfo                 Csv file with optional Comments and UploadedFilename of attachments. Required fields:'Location,FileName,Comments'.

  =============================== Export options.
  -Export                         Path of folder or ZIP archive to receive the exported attachments
  -Overwrite                      Allow overwriting existing files. [default: False]
  -ExportLocations                Locations to export. If no locations are specified, all locations are exported.

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.
```