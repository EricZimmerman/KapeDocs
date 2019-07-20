# Tips and tricks 

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

 
