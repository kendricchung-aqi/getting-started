## QuartZilla

Download the [latest EXE here](https://github.com/AquaticInformatics/getting-started/releases/tag/QuartzZilla)

This console utility can force a Quartz job (AqAutomationService job) to run immediately.

It must be run directly on the AQTS app server.

Like many of the AQTS console utilities, this tool:
- Requires the .NET 4.7 runtime, which is pre-installed on all up-to-date Windows 7+ systems.
- Is a single EXE, and can be run from any directory.
- It will create a `QuartzZilla.log` log file in the same folder as the EXE.
- It supports the flexible [`@options` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) for when the command line gets a bit daunting.
- See the `/help` option for more details.

### Show all the job names on an app server

Just run the tool without any arguments to list all the Quartz job names detected on the app server.

```sh
$ QuartzZilla
01-19 22:40:34.611 INFO  - Connecting to local AQTS app server ...
01-19 22:40:35.363 INFO  - Connected to 2021.4.72.0
01-19 22:40:35.363 INFO  - Fetching automation settings ...
01-19 22:40:35.566 INFO  -
01-19 22:40:35.627 INFO  - 9 jobs detected:
01-19 22:40:35.643 INFO  - ================
01-19 22:40:35.643 INFO  - EventEngine.CompletedWorkUnitCleanup
01-19 22:40:35.643 INFO  - EventEngine.FailedWorkUnitCleanup
01-19 22:40:35.643 INFO  - EventLog.Cleanup
01-19 22:40:35.643 INFO  - OpenIDConnect.ExpiredNonceCleanup
01-19 22:40:35.643 INFO  - PluginRegistry.UnusedPluginCleanup
01-19 22:40:35.643 INFO  - Reporting.GeneratedReportCleanup
01-19 22:40:35.643 INFO  - TimeSeriesAppend.Cleanup
01-19 22:40:35.643 INFO  - TimeSeriesEventLog.Cleanup
01-19 22:40:35.643 INFO  - UserSession.Cleanup
```

### Force a Quartz job to run immediately

Specify the name of the Quartz job to run (run the tool without any arguments to see the list of job names).

- By default, the job should start in about 30 seconds.
- The tool must be run directly on the app server, from a shell with Administrative rights.
- AQTS must be installed and running.
- It is safe to run the tool on a production system.

Running the tool will:
- Use the Provisioning API to temporarily change the start time and interval settings of the named job to start 30 seconds from now.
- Restart the automation service, to force the temporary settings to take effect.
- Monitor the `AqAutomationService.log` for evidence that the job has actually executed.
- Restore the original start time and interval settings for the job (even if something goes sideways)
- Restart the automation service, so it resumes running with the original settings.

```sh
$ QuartzZilla PluginRegistry.UnusedPluginCleanup
01-20 00:05:13.957 INFO  - Connecting to local AQTS app server ...
01-20 00:05:14.677 INFO  - Connected to 2021.4.72.0
01-20 00:05:14.677 INFO  - Fetching automation settings ...
01-20 00:05:14.911 INFO  - Waiting for AquariusAutomationService status Stopped ...
01-20 00:05:15.209 INFO  - Detected AquariusAutomationService Stopped in 276 milliseconds
01-20 00:05:15.209 INFO  - Setting 'PluginRegistry.UnusedPluginCleanup' to run 30 seconds from now at 00:05:44 ...
01-20 00:05:15.239 INFO  - Waiting for AquariusAutomationService status Running ...
01-20 00:05:21.663 INFO  - Detected AquariusAutomationService Running in 6 seconds, 418 milliseconds
01-20 00:05:21.663 INFO  - Waiting for 'PluginRegistry.UnusedPluginCleanup' to complete ...
01-20 00:05:22.067 INFO  - No matches of 'UnusedPlugins: Removed \d+ unused .+ plugins' in 45 new service log lines ...
01-20 00:05:27.477 INFO  - No matches of 'UnusedPlugins: Removed \d+ unused .+ plugins' in 47 new service log lines ...
01-20 00:05:32.864 INFO  - No matches of 'UnusedPlugins: Removed \d+ unused .+ plugins' in 47 new service log lines ...
01-20 00:05:38.240 INFO  - No matches of 'UnusedPlugins: Removed \d+ unused .+ plugins' in 47 new service log lines ...
01-20 00:05:43.646 INFO  - No matches of 'UnusedPlugins: Removed \d+ unused .+ plugins' in 47 new service log lines ...
01-20 00:05:54.100 INFO  - 'PluginRegistry.UnusedPluginCleanup' has completed after 32 seconds, 434 milliseconds.
01-20 00:05:54.100 INFO  - Waiting for AquariusAutomationService status Stopped ...
01-20 00:05:54.364 INFO  - Detected AquariusAutomationService Stopped in 262 milliseconds
01-20 00:05:54.364 INFO  - Restoring original 'PluginRegistry.UnusedPluginCleanup' job settings of 01:00:00 ...
01-20 00:05:54.381 INFO  - Waiting for AquariusAutomationService status Running ...
01-20 00:06:00.788 INFO  - Detected AquariusAutomationService Running in 6 seconds, 393 milliseconds
01-20 00:06:00.788 INFO  - 'PluginRegistry.UnusedPluginCleanup' job ran successfully in 45 seconds, 873 milliseconds.
```

## The `/help` screen

```
Purpose: Force a Quartz job to run right away

Usage: QuartzZilla [-option=value] [@optionsFile] ...

Supported -option=value settings (/option=value works too):

  -JobName               The name of the Quartz job to execute
  -JobStartInterval      Start the job at the current clock plus this interval. [default: 00:00:30]
  -SettleTime            The time to wait after the Quartz job has run [default: 00:00:05]
  -LogScanInterval       Scan the automation service log at this interval. [default: 00:00:05]
  -ServiceWaitTimeout    Timeout for changes in the automation service status. [default: 00:01:00]
  -JobCompletionTimeout  Timeout for a Quartz job to complete [default: 00:05:00]
  -WaitForRegex          Overrides the regular expression used to detect the job completion in the automation log

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.
```