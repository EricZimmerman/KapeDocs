# Frequently asked questions

## Is there a way to have KAPE download all the binaries that are needed for the modules?

In short, no, and there will not ever be a way to do this, as it is a massive liability.

While certain modules come with the binaries they need, in general, the user needs to pick which modules they want to run, they need to take steps to discover and download the binaries, they need to extract and copy them to the proper location.

This way, there is ZERO surprises when KAPE does something.

## How can I execute all of the config files found in a subdirectory of the Targets or Modules directory?

One option is to create a compound target or modules that references all the files in the directory you want to include, but an easier way is to simply include the name of the directory in the appropriate option.

Assuming the directory Targets\Windows exists, you could execute all targets under that directory like this:

`.\kape.exe --tsource c --tdest C:\temp\tout\ --tflush --target Windows`

You can also of course include other target names as well, separating them with a comma:

`.\kape.exe --tsource c --tdest C:\temp\tout\ --tflush --target Windows,Chrome`

## I am in a corporate environment and want to push to S3, but there is a proxy in the way. Does KAPE support this?

it sure does! See [here](https://github.com/EricZimmerman/KapeFiles/issues/121#issuecomment-524682298) for details on how to do this.

