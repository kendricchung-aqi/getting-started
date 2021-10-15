Download the [latest SystemSizer.exe here](https://github.com/AquaticInformatics/getting-started/releases/SystemSizer)

### Allow outbound TCP connections on port 22 to `sftp.aquaticinformatics.com` and everything will be automatic.

The simplest thing you can do is allow your app server to connect to the Support Team's SFTP server to make the secure file transfer.

# SystemSizer - Capture the last week of AQUARIUS product activity for analysis

`SystemSizer.exe` is a console tool used to troubleshoot AQUARIUS platform products.

The tool performs a number of inspections on your systems running AQUARIUS products:
- Analyzes basic sizing, configuration, and AQTS EventProcessor performance.
- Collects relevant logs from all products into a single ZIP archive for easy forwarding to the AI Support Team.
- Can automatically upload the ZIP to the AI SFTP server, if your app server is allowed to connect to the internet.
- Can be run while other AQUARIUS software is running. No scheduled downtime is required.
- Works with AQUARIUS Time-Series (client and server, 3.x and 201x versions), WebPortal, Connect, DAS, Forecast, and EnviroSCADA.

## Why is it named "SystemSizer"?

This tool started its life as a simpler tool to analyze EventProcessor performance, and allow you to correctly **size** your **system**.

Over time, more support features were bolted on, and it has now become the one-stop-solution to quickly allow our Support Team to see relevant activity for the last week and help you figure out what might be going on.

A better name might now be "LogZipper.exe", but we are stuck with the original name. :)

## Usage instructions

**1) Download [SystemSizer.exe](https://github.com/AquaticInformatics/getting-started/releases/download/SystemSizer/SystemSizer.exe) to your app server**

**2) Unblock the downloaded EXE**

Since the file is downloaded from the internet, you may need to explicitly unblock the EXE, since it requires elevated privileges for some of its logfile collection activity.

![Unblock the EXE](https://raw.githubusercontent.com/AquaticInformatics/getting-started/master/images/UnblockSystemSizer.png)

**3) Double-click the SystemSizer.exe from Windows Explorer and follow the prompts.**

If your app server can reach the internet, a ZIP of all the relevant logs will be automatically uploaded to the AI SFTP site and the Support Team will be notified.

### My app server is blocked from the internet. What do I do?

- Allow outbound TCP connections on port 22 to `sftp.aquaticinformatics.com` and everything will be automatic.

That's the simplest thing you can do is allow your app server to connect to the Support Team's SFTP server to make the secure file transfer. Then you won't need to keep reading.

### My IT department won't allow any external internet access from our app server.

That's OK. It is common for AQUARIUS app servers to be blocked from the internet by many IT departments.

When SystemSizer is unable to connect to the AI SFTP site, it just leaves the ZIP file on disk, in the same folder as the SystemSizer.exe. The ZIP file will be named **SystemSizer-_{MachineName}_-_{yyyyMMddHHmmss}_.zip**, substituting you app server's name and the current time.

You will need to send the ZIP file to the AI Support Team somehow. There are a few ways to do this:

**1) On an internet-capable computer, put the ZIP and EXE into the same folder and double click the EXE.**

An "Internet-capable computer" in this context is one which is allowed to make outbound TCP connections on port 22 to `sftp.aquaticinformatics.com`.

In locked-down environments, application servers are often blocked from any internet access, but some client computers are typically allowed to browse the web in some form.

When **SystemSizer.exe** starts, it will attempt to upload any unsent ZIPs in the same folder. This feature allows you to quickly "leap frog" between systems to exfiltrate the ZIP to the AI Support team.

Here is the SystemSizer ZIP and EXE on my app server with no internet connectivity:

![ZIP on blocked app server](https://raw.githubusercontent.com/AquaticInformatics/getting-started/master/images/SystemSizeOnFirewallAppServer.png)

I can use Windows Explorer to browse to that folder over my local network from my internet-capable desktop:

![ZIP via local network](https://raw.githubusercontent.com/AquaticInformatics/getting-started/master/images/BrowseToAppServerFolder.png)

Now just double-clicking the **SystemSizer.exe** from Windows Explorer will automatically start uploading the ZIP from my laptop (which **is* internet connected) to the AI SFTP site. 

![ZIP uploading](https://raw.githubusercontent.com/AquaticInformatics/getting-started/master/images/SystemSizerUploadingFromLaptop.png)

**2) Manually send the ZIP to AI using email or FTP**

You can always send the ZIP to AI support using other file sharing methods. Your organization may already have a way to share large files with Aquatic Informatics.