Download the [latest AttachmentUploader.exe here](https://github.com/AquaticInformatics/getting-started/releases/AttachmentUploader)

# AttachmentUploader

This console utility can be used to upload many attachments to an AQTS system.

- A single EXE .NET console utility for uploading many attachments at once.
- Files to upload are organized by folders named as location identifiers.
- ZIP archives containing files to upload can also be used.
- Filenames containing date patterns are assumed to be field visit attachments for the same date
- Otherwise files will be uploaded as location attachments.
- This tool supports the flexible [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) for when the command line gets a bit daunting.

Some convenience features are available:
- An `AttachmentUploader.log` file is created as a record of the activities performed.
- Defaults to a "dry-run" option, so you can see which files would be uploaded as location or field visit attachments.
- If an attachment of the same name and file size exists at that location or field-visit, it is not re-uploaded. This allows a failed upload attempt to be quickly resumed after correcting any errors.
- Configurable location aliases are supported.
- Large video attachments can automatically be converted to MP4 video files before uploading. This tends to save disk space and yields a better browser playback experience.

## Using the Dry-run option

The `/DryRun=true` option (or its `/N` shortcut) can be used to see how the `AttachmentUploader` will interpret all the files being processed for uploading.

This is the default behaviour of the tool, so that you don't start accidentally uploading files to unexpected landing spots.

This option is useful when debugging the layout of files or ZIP archives being uploaded.
You can inspect the `AttachmentUploader.log` for `WARN` messages, and resolve any complaints before trying the uploads for real.

|`WARN` message pattern|Resolution|
|===|===|
|Skipping unknown location identifier '*path*'<br/>Skipping file '*path*' with no location folder context. | Add a [location alias](#location-identifier-aliases). |
|No field visit time extracted from '*path*'. | Add a [`/DateTimeFormat`](#specifying-field-visit-datetime-patterns-in-filenames) option. |
|Can't find existing visit at *date* for '*path*' | Upload the missing visit and try again. |
|Skipping big file '*path*' | None. |

## Turn off Dry-run mode once you are ready to start uploading files.

Once you are happy with how the tool will categorize your attachments, you can start uploading attachments to AQTS.

Use the `/DryRun=false` option (or its `/Y` shortcut) to disable the dry-run and start uploading attachments.

## What if I accidentally upload files I shouldn't have?

Well, this is embarassing, isn't it?

You can either:
- Restore the DB **and** BLOB storage backups which you absolutely, most definitely did make before you started trying to upload thousands of files all at once.
- Use the Springboard UI to delete each individual file that you accidentally uploaded.

It's your choice.

## How fast will files be uploaded?

The speed at which attachments are uploaded to AQTS will vary based on client, server, and network latency.

But internal testing has seen the tool upload more than 7000 attachments with a total size of over 4 GB in about 7 minutes. So that is reasonably quick.

Go for a coffee but don't go home for the day. :)

## How should the files be organized for speedy uploading?

The `/Root=path` option specifies the root path to inspect for attachment candidates.

- Under the root path, folders should be named with AQTS location identifiers (case-insensitive).
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

By default, the `/Root=path` path folder is expected to contain folders named with AQTS location identifiers.

But locations are often renamed during migration, and legacy attachments are often extracted into ZIP archives with their legacy identifier values.

Use the `-LocationAlias=alias:identifier` option to create an alias from a legacy system to a current AQTS location identifier.

The `-LocationAlias=` prefix is optional.
You can just specify "alias:identifier" as a positional command line option, and the tool will recognize it as an alias.

Using the [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) is a handy way to specify multiple aliases.

If the `C:\FilesToUpload\AllTheFiles.zip` archive from Example 3 was produced by some legacy system, and the true AQTS locations where 1111 and 2222, then this command line would work:

```cmd
C:\> AttachmentUploader.exe -server=myAppServer -Root=C:\FilesToUpload @AllOptions.txt
```

Where `AllOptions.txt` contained these 3 lines (note the use of positional arguments):
```
# Add all the location aliases here, in alias:identifier form. Whitespace around the colon is OK.
Loc1 : 1111
Location 2 : 2222
```

Then the files would be uploaded to the 1111 and 2222 locations, instead of seeing `WARN` statements that 'Loc1' and 'Location 2" are unknown locations.

## Converting video files to MP4

Many common video file formats are quite large. (AVI files and MOV files from phones are generally large and uncompressed). AQTS has a maximum file upload size of 100 MB per file. It is very easy for uncompressed video to exceed this limit.

The uploader tool can detect these video formats (and others) and use the `ConvertToMP4.exe` tool to convert these larger files into compressed MP4 video files, which tend to be 4x-10x smaller for the same resolution, and play back nicely in all modern browsers.

Set the `/ConvertToMP4Path=somePath` option to point to the `ConvertToMP4.exe` tool, and the `AttachmentUploader` will automatically convert every video file to an MP4, and then upload the MP4 instead of the larger AVI/MOV. You will avoid most file upload size errors and not notice any drop in video/audio quality.

If no `/ConvertToMP4Path` option is set, the same directory as `AttachmentUploader.exe` is searched.

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

The `AttachmentUploader` tool tries to detect field visit attachments using 3 rules and two command line options.

- The `/FieldVisitSubFolders=name1,...,nameN` list specifies folders which contain field visit files.
- The `/DateTimeFormats=pattern1,...,patternN` list specifies patterns to extract timestamps from filenames.

These list options have some common behaviour:
- Each list has some reasonable defaults.
- To reset the list, just specify an empty value after the equals sign. (so `/DateTimeFormats=` will discard the default datetime patterns)
- Each option can be specified multiple times, and any values are added to the current list.
- Using an `@options.txt` file usually helps keep the command line sane.

The 3 rules for matching field visit attachments are:

### Rule 1) Does the file path match a `/FieldVisitSubFolders` name?

- Look at the file path to see if contains a folder exactly matching one of the expected names (which defaults to a single name of "Visits").
- More than one folder name can be specified by using a comma-separated list. `/FieldVisitSubFolders=Visits,Measurements` names.
- Folder name matching is case-insensitive.

### Rule 2) Does the filename match a `/DateTimeFormats` pattern?

- Look at the filename (but not the path) and see if one of the `/DateTimeFormats` patterns is matched.
- If no datetime pattern is matched, a `WARN` message is logged.

### Rule 3) Does the datetime extracted from the filename match an existing field visit?

- If a timestamp is extracted, and a visit exists at the location, then the file is uploaded as a visit attachment.
- If a timestamp is extracted but no matching visit is found, a `WARN` message is logged.

### The `/UploadUnknownVisitAttachments=true` option can force unmatched visit attachments to their location

When the `/UploadUnknownVisitAttachments` option is set, any files failing rule 2 or rule 3 will still be uploaded as a location attachment.

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

## Uploading attachments with comments and tag values

The `-AttachmentInfoCsvFile=path` option can be used to upload attachments with a specific comment or tag value.

## Exporting existing attachments

The tool can also export existing attachments from AQTS into a folder or ZIP file, which could be used to re-import the attachments into another AQTS system with the same locations and visits.

- `-Export=folder` will export the attachments to a folder, with a location subfolder for each exported location.
- `-Export=file.zip` will export the attachments to a ZIP file, to conserve storage space.
- Use the `-ExportLocations=loc1,loc2` option to specify a subset of location identifiers to export, otherwise all locations will be exported.

## Help screen

```
Upload attachments to an AQTS system.

Usage: AttachmentUploader [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  =============================== Upload attachments to an AQTS system.
  -Server                         The AQTS app server
  -Username                       AQTS username [default: admin]
  -Password                       AQTS credentials [default: admin]

  =============================== Attachment processing options.
  -Root                           The root folder containing attachments organized by location identifier folders.
  -InspectZipArchives             Inspect ZIP archives for attachments [default: True]
  -DryRun                         Don't upload the attachments. Use /N or /Y as a shortcut for /DryRun=True or /DryRun=False [default: True]
  -FieldVisitSubFolders           Folder names indicating field visit attachments [default: Visits]
  -DateTimeFormats                Datetime formats to match in filenames. [default: yyyy-MM-dd]
  -UploadUnknownVisitAttachments  Upload unknown visit attachments as location attachments instead. [default: False]
  -IgnoreFiles                    Filenames to ignore [default: .ppinfocache, Thumbs.db, Desktop.ini, .DS_Store]
  -UploadLimitMB                  Upload file size limit in megabytes [default: 100]
  -ConvertToMP4Path               Path to the ConvertToMP4.exe utility, used to convert videos to MP4 format before upload.
  -LocationAlias                  Add location aliases in LocationAlias=alias:locationIdentifier form

  =============================== Options for uploading files with comments and tags.
  -AttachmentInfoCsvFile          Csv file with comments and tags of attachments. Required fields:'Location,Comments,FileName,Tags'.
                                  Tag separator is '|'
  -UsePathInCsvFile               When true, use 'FileName' field in 'AttachmentInfoCsvFile' to locate attachment files.
                                  If [FileName] does not start with '\' or is not fully qualified, root folder will be added.
                                  Default is false: search root and sub folders for attachment files. [default: False]

  =============================== Export options.
  -Export                         Path of folder or ZIP archive to receive the exported attachments
  -Overwrite                      Allow overwriting existing files. [default: False]
  -ExportLocations                Locations to export. If no locations are specified, all locations are exported.

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.
```