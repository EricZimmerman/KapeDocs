# Tips and tricks 

## Quick SFTP server standup HOWTO

See [here](https://medium.com/@bromiley/automating-sftp-creation-for-kapes-sake-b0bc68d10522) for a nice write-up by Matt Bromiley on how to use Digital Ocean to quickly stand up SFTP for use with KAPE

## Automating KAPE at Scale

From Brian Maloney:

This setup will give you a remote KAPE instance with automatic parsing and email alerting. More info can be found [here](https://malwaremaloney.blogspot.com/2020/06/kape-at-scale.html)

From Mark Hallman:

Use ZeroTier and KAPE for remote collection magic! Read all about it ![here](External\Remote_Collections_KAPE\Remote Collections with KAPE.md)

## Using environment variables to create unique destinations

In some situations, when collecting multiple machines at once to a common location for example, you will want to keep things separated by machine. You can use environment variables in KAPE to achieve this. 

In a CMD shell, it would look like this (for simplicity, only --mdest is shown):

![Target example](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/cmdEnvVar.jpg)

And in PowerShell, like this:

![Target example](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/psEnvVar.jpg)

Adjust the environment variable as needed to best suit your needs!


## Running KAPE as SYSTEM vs administrator

From Troy Larson:

While running tasks, such as sigcheck, through Kape, as administrator, I encountered fewer file access errors than running the same tasks as administrator outside of Kape.  However, with Kape, there were still file access errors.


The only way that I could run tasks without file access errors was to run tasks as system, in or outside of Kape. 

Summary:

Running tasks as administrator through Kape has fewer file access errors than running the same tasks as administrator outside of Kape.  Running the same tasks as system, however, had no file access errors, either in or outside of Kape.

If my testing is correct, then Kape should best be run as system.

I used psexec.exe to run Kape as system: e.g., PsExec64.exe -s E:\kape\Kape.exe --msource G: --module MSFTallScan --mdest E:\kape\parsed

## Using KAPE with CrowdStrike Falcon (Real Time Response)

From Martin Willing:

Introduction: 
With the Real Time Response (RTR) feature of CrowdStrike Falcon (Endpoint Detection & Response platform) you can deploy files to live endpoints and run custom scripts. You can connect / start a session with a live endpoint (Shell with a set of built-in commands) during a security investigation. On one hand you can use multiple RTR commands (Sensor features) to interact with the live endpoint and on the other hand you can also use custom scripts (built on top of Windows PowerShell).

You can use all command line EZTools and especially kape.exe as low level file extractor on NTFS volumes to collect evidence.

Usage: 

1. I always start with creating a temporary working directory on the live endpoint and change my directory afterwards.

   ```
   C:\> mkdir "C:\RTR"
   C:\> cd "C:\RTR"
   C:\RTR> 
 ```

2. After moving to my working directory I deploy 7za.exe (7-Zip standalone binary) and KAPE as a custom package (only the files I need for a specific task).

   ![01](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/csFalcon01.png)
   ![02](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/csFalcon02.png)
   ![03](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/csFalcon03.png)

   ```
   C:\RTR> put "7za.exe" 
   C:\RTR> put "MFT-Collector.7z" 
   C:\RTR> ls 
   ```

   Notes: All my files and response scripts are stored in the CrowdStrike cloud, so that I can deploy them also when an endpoint is isolated (via Network Containment feature). You have to use the file names and script names used in CrowdStrike cloud.

3. Then I run my response script (MFT-Collector.ps1)

   ```
   runscript -CloudFile="MFT-Collector" -CommandLine="" 
   ```

   or

   ```
   runscript -CloudFile="MFT-Collector" -CommandLine="EnableVSC" 
   ```

   Note: Custom scripts support only one parameter. I like to use this parameter as an optional parameter to enable collection of evidence stored in Volume Shadow Copies.

   My PowerShell script extracts the files from the custom package (MFT-Collector.7z) and KAPE collects the targeted evidence ($MFT).

4. With the RTR command 'zip' I create a file of my output directory and with the RTR command 'get' I collect the ZIP archive.

   Note: The maximum file size for the collection via 'get' is 4 GB. When needed I split the files with 7za.exe (e.g. big RAM files).


Links:
https://www.crowdstrike.com/endpoint-security-products/falcon-endpoint-protection-enterprise/



 
