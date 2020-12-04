# Useful tips to get started with Nextflow
... that are hidden from the official documentation

### The working directory
- Each parallel execution of a process happens in its own working directory. It is the folder named like `/path_to_tmp/4d9c3b333734a5b63d66f0bc0cfcdc` Nextflow points you to when there is error. One can find the folder path in the .nextflow.log or in the report.html.
- This folder contains symlinks for the input files (see next point). 
- This folder will also contains all output files, and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`.
- Knowing this, if there is `cd` in the script section, it will leave the working directory. The output files may get written to the folder that was `cd` into, but Nextflow will still search for output files in working directory to put in `publishDir` and will not be able to find them. 

### What really happens with path("A.txt")
- `path(A)` is the same as `file(A)`. `tuple` is the same as `set`. It's recommended to use `path` and `tuple` with newer versions.
- `Channel.from( "path/A.txt" )` will put `path/A.txt` as is into the channel, whereas `Channel.fromPath( "path/A.txt" )` will create a symlink named `A.txt` linking to `path/A.txt` into the working directory. 
- This is the same as `val("A.txt")` and `file("A.txt")`. When one declares `input: file("A.txt")`, Nextflow actually creates a symlink named `A.txt` linking to `path/A.txt` in the working directory, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.

### path("A.txt") vs. path(A)
- With `input: path("A.txt")` one can refer to it in the script as `A.txt`
- With `input: path(A)` one can refer to A in the script as `$A`
- `path( )` and `path " "` are generally exchangale. Exception is (tip from [@danielecook](https://github.com/danielecook)): 
  - if not in a tuple, use `input: path “input.tsv”` 
  - if in a tuple, use `input: tuple path(“input1.tsv”), path(“input2.tsv”)`
  - this is one of the mysterious errors hopefully will get fixed soon.

### DLS2 vs DLS1
- In DSL1, each channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes, but each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module.
- DSL2 also enforce that each process takes only 1 input channel, so all inputs needs to be combined into 1 channel. See **Nextflow_cheatsheet_channel**.

### Nextflow reports
- Having `report.enabled = true` and `timeline.enabled = true` in the config will let Nextflow write out report for the run. They will contain resource usage, status for each execution, path to working directory and time spent queueing and running for each step. Extremely useful for troubleshooting and optimize resources.
