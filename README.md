# Useful tips to get started with Nextflow
**(Please report errors in Issues. Thank you!)**

### The working directory
- **Each execution of a process happens in its own temporary working directory.** 
- It is the folder named like `/path_to_tmp/4d9c3b333734a5b63d66f0bc0cfcdc` that Nextflow points you to when there is error. One can find the folder path in the .nextflow.log or in the report.html. This folder contains the error log that could be useful for debugging.
- This folder only contains files (usually in form of symlinks, see below) from the input channel, so it's isolated from the rest of the file system. 
- This folder will also contain all output files (unless specifically directed elsewhere), and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`.
- This is imporatnt to keep in mind when the `"""` script section `"""` involves changing folders, such as:
  - `cd` will go outside of the working directory and can no longer find files from the input channels. The output files will get written to the folder that was `cd` into, so Nextflow will not be able to find output files in working directory to put in `publishDir`. 
  - with `rmarkdown::render( knit_root_dir = "folder/" )`, rmarkdown will knit, look for input files and write out files in respect to the `knit_root_dir` (which defaults to location of the .rmd file), but the markdown report itself (.pdf or .html) will be generated where the .rmd is. Among them, only files in Nextflow working directory can go to output channel. 
- Run `nextflow clean -f` in the excecution folder to clean up the working directories.

### The relative path to Nextflow
- Throughout Nextflow scripts, one can use 
  - `${workflow.projectDir}` to refer to where the nextflow script (usually main.nf) locates. For example `Rscript ${workflow.projectDir}/bin/task.R`will point to the R script in the bin folder where the nextflow script is.
  - `${workflow.launchDir}` to refer to where the script is called from. 

### path("A.txt")
- `Channel.from( "A.txt" )` will put `A.txt` as is into the channel 
- `Channel.fromPath( "A.txt" )` will add a path (usually current directory) and put `/path/A.txt` into the channel. 
- `Channel.fromPath( "/path/A.txt" )` will put `/path/A.txt` into the channel. In other words, `Channel.fromPath` will always include a path to the file.
- This goes hand in hand with `input: path("A.txt")` inside the process declaration, where **Nextflow actually creates a symlink named `A.txt`** (note the path is stripped) **linking to `/path/A.txt` in the working directory**, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.
- `path(A)` is the same as `file(A)`. `tuple` is the same as `set`. It's recommended to use `path` and `tuple` with newer versions.

### path("A.txt") vs. path(A)
- With `input: path("A.txt")` one can refer to the file in the script as `A.txt`. Side note `A.txt` doesn't have to be the same name as in channel creation, it can be anything, `B.txt`, `name` etc. 
- With `input: path(A)` one can refer to A in the script as `$A`
- `path( )` and `path " "` generally both work. Exception is (tip from [@danielecook](https://github.com/danielecook), tested in Nextflow 20.01.0): 
  - if not in a tuple, use `input: path “input.tsv”` 
  - if in a tuple, use `input: tuple path(“input1.tsv”), path(“input2.tsv”)`

### DLS2 vs DLS1
- In DSL1, each channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes, but each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module.
- DSL2 also enforce that each process takes only 1 input channel, so all inputs needs to be combined into 1 channel. See **Nextflow_cheatsheet_channel**.

### Nextflow reports
- Having `report.enabled = true` and `timeline.enabled = true` in the config will let Nextflow write out report for the run. 
- They contain resource usage, status for each execution, path to working directory and time spent queueing and running for each step. 
- Extremely useful for troubleshooting and optimize resources.
- `dag.enabled = true` will draw a flowchart for the pipeline. Needs graphviz installed.


