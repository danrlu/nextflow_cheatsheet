# Useful tips to get started with Nextflow
... that are hidden from the official documentation

### The working directory
Each parallel execution of a process happens in its own working directory (the folder with a random name Nextflow points you to when there is error). This folder contains symlinks for the input files (see next point). This folder will also contains all output files, and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`

### val(A) vs. file(A)
`Channel.from( 'path/A' )` will put `path/A` as is into the channel, whereas `Channel.fromPath( 'path/A' )` will create a symlink named `A` linking to `path/A` into the working directory. This is the same as `val(A)` and `file(A)`. When one declares `input: file(A)`, Nextflow actually creates a symlink for `A` in the working directory, so it can be accessed within the working directory by the script `cat $A` without a path


### DLS2 vs DLS1
In DSL1, each channel can only be used once. In DSL2, a channel can be fed into multiple processes, but each process can only be called once. If a process needs to run on 2 input channels, either combine them into 1 channel so a parallel processes runs for each of them, or put the process in a module and import module.
