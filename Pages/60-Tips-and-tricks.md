# Tips and tricks 

## Quick SFTP server standup HOWTO

See [here](https://medium.com/@bromiley/automating-sftp-creation-for-kapes-sake-b0bc68d10522) for a nice write-up by Matt Bromiley on how to use Digital Ocean to quickly stand up SFTP for use with KAPE

## Automating KAPE at Scale

### From Brian Maloney:

This setup will give you a remote KAPE instance with automatic parsing and email alerting. More info can be found [here](https://malwaremaloney.blogspot.com/2020/06/kape-at-scale.html)

### From Mark Hallman:

Use ZeroTier and KAPE for remote collection magic! Read all about it [here](External\Remote_Collections_KAPE\Remote Collections with KAPE.md)

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


## KAPE Target Creation

### From Andrew Rathbun:

#### KAPE Target Creation Tips and Tricks

When it comes to creating a KAPE Target, I strongly recommend installing [Voidtools' Everything](https://www.voidtools.com/) prior to beginning this process. Not only should you be using this tool in your everyday Windows-computing life instead of Windows Search, but it makes locating where artifacts are stored MUCH easier. 

As you're installing an application that you want to do more research into, open Everything and sort on Last Modified. That way, you can see files added to your computer in real-time as the program is installing. Once the program is installed, keep Everything opened and watch which files are updated as you use the program. If nothing is happening, fully exit the program (not minimize or close to taskbar) and observe what files are modified. Often, these are what you want to look further into as they will likely be writing data that can serve to be an important artifact for that program!

Also, once you install an application, do a search either for the name of the application, the developer, or anything that might populate results in Everything for where data could possibly be stored for this application. The reason why I say to get creative with your search terms is because not every application stores data within `.\AppData\Local\ApplicationName\`. For instance, Directory Opus stores data in a folder called GPSoftware. Edit Pad Pro and other apps created by that developer stored all files in a folder call JGsoft, which is short for Just Great Software. However, some developers are nice and just searching the application's name will yield everything there is to find.

You will notice some applications store data in `.\AppData\Local\`, some store in `.\AppData\Roaming\`, some store in both, some store in neither and rather store in `C:\ProgramData\`, some store in all three, and some store in none of those and choose some location like a user's Documents folder. It's MUCH easier to search in Everything and focus on the Path column to see where artifacts reside on disk. See the example below:

![Voidtools Everything Search Example](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/VoidtoolsEverythingSearchExample.gif)

In the GIF above, you can see that Everything stores a .ini file in the `.\AppData\Roaming\` folder and a .db file in the `.\AppData\Local\` folder. That gives me a pointer to check out those locations for those artifacts and potentially more!

Everything makes your life easy in that when you find a file you may want to look at, just right click on the file and Open Path. That way, you'll be brought to the folder where that file resides and you can see not only the file of interest but other potential files and folders of interest. 

![Voidtools Everything Right Click Open Path](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/Pictures/VoidtoolsEverythingRightClickOpenPath.gif)

Once you're in the folder of interest, dive into each file and subfolder and see what is readable in an advanced text editor like Notepad++, Edit Pad Pro, Sublime Text, etc. Regardless of the file extension of the artifacts you find, drag and drop it into your favorite text editor and find useful data that is human readable. If it's human readable and you think there could be value to an investigation, private/LE/otherwise, then create a target for it!

When you're creating Targets, it's not mandatory but it's appreciated if you include some comments below as to what the user can reasonably expect to get by using the Target you created. Any direction you can give them could potentially help a very important investigation down the road so the more you do on the front end, the more an investigator and the victim/client they're working for will benefit on the back end. By creating the Target, YOU are the subject matter expert on that artifact because you're doing the legwork for everyone else, so while you're in the trenches, report on what you see as verbose as possible. That way, you won't have to go back there again because you did it right the first time!

#### Need Ideas for Targets?

Are you looking at the current Targets for KAPE and think there's no possible way there's any applications left to research and create a Target for? If this is your thought process, stay a while and listen! 

[AlternativeTo](https://alternativeto.net/) is an amazing site where you can put in a popular application like Notepad and search for alternatives to that program. They are sorted by community-driven upvotes which will show you the most popular applications used by the community. The most popular ones, in my humble opinion, should have Targets created for them in KAPE at some point or another given that there is a larger chance a suspect/client could have artifacts from these applications that could help an investigation reach a favorable outcome. Now, every application that's ever existed doesn't need a KAPE Target created for it, but if you think the artifacts located for a respective application could help put a child predator (an extreme example) in prison based on the evidence recorded by that application, then a Target should be created. 

Additionally, everyone has heard of or used [Ninite](https://ninite.com/) when it comes time to reformat a computer and get the standard battery of useful programs all in one go. Ninite is extremely popular and therefore so are the applications that are included in their service. These applications change often so it's worth checking back every now and then to see what's new. This is exactly why I created a Target for [SugarSync](https://github.com/EricZimmerman/KapeFiles/blob/master/Targets/Apps/SugarSync.tkape). I had never heard of it before, but it was public-facing on Ninite's site next to OneDrive, Google Drive, and Dropbox. I use all three of those services in my daily life. I don't use SugarSync. I won't use SugarSync. But that doesn't mean someone else/many people won't thanks to Ninite! Now, at least it's covered should someone need it down the road given the visibility of the application provided by Ninite.

Wikipedia also typically has nearly exhaustive lists of types of programs that can give ideas for potential Targets. For example, [Comparison of online backup services](https://en.wikipedia.org/wiki/Comparison_of_online_backup_services) would provide some ideas for cloud storage services in addition to the big players like OneDrive, Google Drive, Dropbox, etc. [List of P2P protocols](https://en.wikipedia.org/wiki/List_of_P2P_protocols) would provide some ideas for more P2P clients, which of course are common in child exploitation cases as well as IP theft cases. Remember, just because a program exists doesn't mean a Target needs to exist for it. Be reasonable! Also, just because a program exists doesn't mean it stores anything unencrypted or readable with a text editor or other readily available tool. Do your research before blindly creating Targets for every program under the sun!

#### Conclusion
If you have any questions on how to create a Target or need some feedback on an idea for a Target, please do not hesitate to reach out to me. I can always be found on the [Digital Forensics Discord Server](https://aboutdfir.com/a-beginners-guide-to-the-digital-forensics-discord-server/). 
