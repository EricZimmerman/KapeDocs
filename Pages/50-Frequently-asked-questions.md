# Frequently asked questions

## How can I execute all of the config files found in a subdirectory of the Targets or Modules?

One option is to create a compound target or modules that references all the files in the directory you want to include, but an easier way is to simply include the name of the directory in the appropriate option.

Assuming the directory Targets\Windows exists, you could execute all targets under that directory like this:

`.\kape.exe --tsource c --tdest C:\temp\tout\ --tflush --target Windows`

You can also of course include other target names as well, separating them with a comma:

`.\kape.exe --tsource c --tdest C:\temp\tout\ --tflush --target Windows,Chrome`