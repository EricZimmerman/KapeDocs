<p align="center">
  <img  src="https://ericzimmerman.github.io/logoSmall.jpg">
</p>

|Forensic tools  |Version| Purpose| 
|--|--|--
| AmcacheParser | [1.3.3.0](https://f001.backblazeb2.com/file/EricZimmermanTools/AmcacheParser.zip) | Amcache.hve parser with lots of extra features. Handles locked files
| AppCompatCacheParser | [1.4.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/AppCompatCacheParser.zip)| AppCompatCache aka ShimCache parser. Handles locked files
| bstrings | [ 1.5.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/bstrings.zip)| Find them strings yo. Built in regex patterns. Handles locked files
| EZViewer | [0.5.4.0](https://f001.backblazeb2.com/file/EricZimmermanTools/EZViewer.zip)| Standalone, zero dependency viewer for .doc, .docx, .xls, .xlsx, .txt, .log, .rtf, .otd, .htm, .html, .mht, .csv, and .pdf. Any non-supported files are shown in a hex editor (with data interpreter!)
| Evtx Explorer/EvtxECmd | [0.4.5.0](https://f001.backblazeb2.com/file/EricZimmermanTools/EvtxExplorer.zip)| Event log (evtx) parser with standardized CSV, XML, and json output! Custom maps, locked file support, and more!
| Hasher | [1.9.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/hasher.zip)| Hash all the things
| JLECmd | [1.3.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/JLECmd.zip)| Jump List parser
| JumpList Explorer | [1.3.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/JumpListExplorer.zip) | GUI based Jump List viewer 
| LECmd  | [1.3.2.0](https://f001.backblazeb2.com/file/EricZimmermanTools/LECmd.zip) | Parse lnk files
| MFTECmd |[0.4.4.2](https://f001.backblazeb2.com/file/EricZimmermanTools/MFTECmd.zip) | $MFT, $Boot, $J, $SDS, and $LogFile (coming soon) parser. Handles locked files
| PECmd  | [1.3.2.0](https://f001.backblazeb2.com/file/EricZimmermanTools/PECmd.zip)| Prefetch parser
| RBCmd  | [0.4.1.0](https://f001.backblazeb2.com/file/EricZimmermanTools/RBCmd.zip)| Recycle Bin artifact (INFO2/$I) parser
| RecentFileCacheParser | [0.7.0.1](https://f001.backblazeb2.com/file/EricZimmermanTools/RecentFileCacheParser.zip) | RecentFileCache parser
| Registry Explorer/RECmd | [1.4.2.0](https://f001.backblazeb2.com/file/EricZimmermanTools/RegistryExplorer_RECmd.zip)| Registy viewer with searching, multi-hive support, plugins, and more. Handles locked files
| SDB Explorer | [0.6.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/SDBExplorer.zip)| Shim database GUI
| ShellBags Explorer | [1.3.2.0](https://f001.backblazeb2.com/file/EricZimmermanTools/ShellBagsExplorer.zip)| GUI for browsing shellbags data. Handles locked files
| Timeline Explorer | [0.9.1.0](https://f001.backblazeb2.com/file/EricZimmermanTools/TimelineExplorer.zip) | View CSV and Excel files, filter, group, sort, etc. with ease
| VSCMount |[0.5.3.0](https://f001.backblazeb2.com/file/EricZimmermanTools/VSCMount.zip) | Mount all VSCs on a drive letter to a given mount point
| WxTCmd | [0.3.1.0](https://f001.backblazeb2.com/file/EricZimmermanTools/WxTCmd.zip) | Windows 10 Timeline database parser

***

|Other tools  |Version| Purpose
|--|--|--
| KAPE | [NA](https://learn.duffandphelps.com/kape?utm_campaign=2019_cyberitbn-KAPE-launch&utm_source=kroll&utm_medium=referral&utm_term=kape-gui-blogpost) | Kroll Artifact Parser/Extractor: Flexible, high speed collection of files as well as processing of files. Many many features
| iisGeoLocate | [1.4.0.0](https://f001.backblazeb2.com/file/EricZimmermanTools/iisGeolocate.zip)| Geolocate IP addresses found in IIS logs
| TimeApp | [NA](https://f001.backblazeb2.com/file/EricZimmermanTools/TimeApp.zip)| A simple app that shows current time (local and UTC) and optionally, public IP address. Great for testing
| XWFIM | [NA](https://f001.backblazeb2.com/file/EricZimmermanTools/XWFIM.zip) | X-Ways Forensics installation manager
| Get-ZimmermanTools | [NA](https://f001.backblazeb2.com/file/EricZimmermanTools/Get-ZimmermanTools.zip) | PowerShell script to auto discover and update everything above.



***

|Other files  |Version| Purpose
|--|--|--
| nlog.config | [NA](https://f001.backblazeb2.com/file/EricZimmermanTools/nlog.config)| Place this in same directory as CLI tools and you can alter the colors used. Good for white background with black font, etc. Do not change anything but the colors.
| Change log | [NA](https://f001.backblazeb2.com/file/EricZimmermanTools/ChangeLog.txt)| 




***
## Requirements and troubleshooting

 - All software requires at least [Microsoft .net 4.6](https://www.microsoft.com/en-us/download/details.aspx?id=48137) or newer! You will get errors running these without at least 4.6. When in doubt, install it!
 - **DO NOT RUN ANYTHING FOUND HERE FROM 'C:\PROGRAM FILES' DIRECTORY** (unless you run them as administrator)!
 - **DO NOT USE WINDOWS TO EXTRACT THINGS.** Use 7-Zip or Winrar as Windows will block the DLLs!
 - All software is digitally signed. Once you verify the signature as coming from me, any anti-virus hits are false positives. When in doubt, download the files directly from here!
 - **If you get DPI scaling issues, make a shortcut (or directly against the exe), edit the properties, then click Compatibility. Under Change high DPI settings, check Override high DPI scaling behavior at bottom and choose System, then click OK out of the dialog**

***
[![Donate](https://ericzimmerman.github.io/Quarter16.png)](https://www.patreon.com/ericzimmerman) **[Patreon](https://www.patreon.com/ericzimmerman)**

[![Donate](https://ericzimmerman.github.io/Quarter16.png)](https://paypal.me/ericrzimmerman) **[Paypal](https://paypal.me/ericrzimmerman)**

Open Source Development funding and support provided by the following contributors: [SANS Institute](http://sans.org/) and [SANS DFIR](http://dfir.sans.org/).
