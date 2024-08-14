# Tips and cheatsheet for Nextflow

These are notes for myself gathered through using Nextflow, and hopefully useful for others. **Error reports and suggestions welcome!**

### Some resources
- [DSL2 beginners' guide](https://github.com/chlazaris/Nextflow_training/blob/main/nextflow_cheatsheet.md) by [Harriz Lazaris](https://github.com/chlazaris) for Nextflow.

- [Nextflow channel operation cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf). 

- [Practical guide](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_convert_DSL2.pdf) to convert DSL1 to DSL2.

- [Nextflow Slack](https://www.nextflow.io/slack-invite.html) for all questions and to connect with others.

### The working directory
Understanding working directory was the hardest learning piece for me, and it turned out to be key to understand where the files are and how to debug errors b/c often all files and logs you need are in the working directory.  
- **Each execution of a process happens in its own temporary working directory.** 
- Specify the location of the parent working directory with `workDir = '/path_to_tmp/'` in nextflow.config, or with `-w` option when running `nextflow main.nf`.
- Each excecution of a process creates one folder in the working directory. This folder starts off with files only from the input channel (usually in form of symlinks, see below), so it's fairly isolated from the rest of the file system. 
- As the process runs, this folder will also contain all intermediate files, logs, and output files (unless specifically directed elsewhere), and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`. 
  - Anything you want to specify in `publishDir` needs to be in an output channel.
  - Note that with `publishDir "path", mode: 'move'`, the output file will be moved away from the working directory and Nextflow will not be able to use it as input for another process, so only use this option when there is not a following process that uses the output file. 
  - Be mindful that if the `""" (script section) """` involves changing directory, such as `cd` or `rmarkdown::render( knit_root_dir = "folder/" )`, Nextflow will still only search the working directory for output files b/c the execution is in the working directory. [tl;dr is this gets tricky, so try let Nextflow handle folder navigation as much as possible. ] -> ? ( not clear): [This is tricky, so let Nextflow handle folder navigation as much as possible].
- To find the location of the working directory: it is the folder named like `/path_to_tmp/4d9c3b333734a5b63d66f0bc0cfcdc` that Nextflow points you to, when there is an error in execution. This folder usually already contains all files needed to reproduce the error, and Nextflow error message gives clear direction how to reproduce the error. One can also find the folder path in the `.nextflow.log` or in the `report.html`. 
- Run `nextflow clean -f` in the execution folder to clean up the working directories, which often gets large unnoticed.


### Where am I?
Actual data is usually elsewhere from where the Nextflow scripts are, and to be able to specify relative file path makes the code more portable. The options below are much more reliable than `$PWD` or `$pwd`.
- In Nextflow scripts (.nf files), one can use 
  - `${workflow.projectDir}` to refer to where the project locates (usually the folder of `main.nf`). For example: `publishDir "${workflow.projectDir}/output", mode: 'copy'` or `Rscript ${workflow.projectDir}/bin/task.R`.
  - `${workflow.launchDir}` to refer to where the script is called from, aka the current folder in Terminal when running `nextflow main.nf`.
- `$baseDir` usually refers to the same folder as `${workflow.projectDir}` but it can also be used in the config file, where `${workflow.projectDir}` and `${workflow.launchDir}` are not accessible.   


### Print - debugger's best friend
The hardest error to debug (assuming one is familiar with bioinformatics tools) is often channels structure TnT
- To print a channel, use `.view()`. It's especially useful to resolve `WARN: Input tuple does not match input set cardinality declared by process`. (Don't forget to remove `.view()` after debugging) 
```
  channel_vcf
    .combine(channel_index)
    .combine(channel_chr)
    .view()
```
- To print from the script section inside the processes, add `echo true`. This is very useful to check whether a channel has passed desired information in correct format to the process.
```
  process test {
    debug true    // this will print the stdout from the script section on Terminal
    input: path(file)
    """
    head $file
    """
  }
```
- The [channel operation cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) contains the channel operations I use most often.


### `Channel.from` and `Channel.fromPath` what's the difference?
As biologists, we turn every rock.
- `Channel.from( "A.txt" )` will put `A.txt` as is into the channel 
- `Channel.fromPath( "A.txt" )` will add a full path (usually current directory) and put `/path/A.txt` into the channel. 
- `Channel.fromPath( "folder/A.txt" )` will add a full path (usually current directory) and put `/path/folder/A.txt` into the channel. 
- `Channel.fromPath( "/path/A.txt" )` will put `/path/A.txt` into the channel. 
- In other words, `Channel.fromPath` will only add a full path if there isn't already one and ensure there is always a full path in the resulting channel.
- This goes hand in hand with `input: path("A.txt")` inside the process, where **Nextflow actually creates a symlink named `A.txt`** (note the path from first `/` to last `/` is stripped) **linking to `/path/A.txt` in the working directory**, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.


### `input: path("A.txt")` in the process section 
- With `input: path("A.txt")` one can refer to the file in the script as `A.txt`. Side note `A.txt` doesn't have to be the same name as in channel creation, it can be anything, `input: path("B.txt")`, `input: path("n")` etc. 
- With `input: path(A)` one can refer to the file in the script as `$A`, and the value of `$A` will be the original file name (without path, see section above). 
- `input: path("A.txt")` and `input: path "A.txt"` generally both work. Occasionally had errors that required the following (tip from [@danielecook](https://github.com/danielecook)): 
  - If not in a tuple, use `input: path "A.txt"` 
  - If in a tuple, use `input: tuple path("A.txt"), path("B.txt")`
  - This goes the same for `output`.
- From [pditommaso](https://github.com/pditommaso): `path(A)` is almost the same as `file(A)`, however the first interprets a value of type string as the input file path (ie the location in the file system where it's stored), the latter interprets a value of type string and materialises it to a temporary files. It's recommended to use `path` since it's less ambiguous and fits better in most use-cases.


### DSL2
This is a little outdated. Is anyone still DSL1-ing??
- Moving to DSL2 is a one-way street. It's so intuitive with clean and readable code.
- In DSL1, each queue channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes
- In DSL2, each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module. (One may be able to call a process in different workflows, haven't tested yet).
- DSL2 also enforces all inputs to be combined into 1 channel before it goes into a process. See the [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) for useful operators. 
- [Simple steps to convert from original syntax to DSL2](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_convert_DSL2.pdf)
- [Deprecated operators](https://www.nextflow.io/docs/latest/dsl2.html#dsl2-migration-notes).


### Run reports
Beautiful graphics especially useful for performance monitoring.
- `nextflow main.nf -with-report -with-timeline -with-dag`
- `-with-report` Nextflow html report contains resource usage for each process, and details (most useful being the status and working directory) for each process. 
- `-with-timeline` How much wait time and run time each process took for the run. Very useful reference for optimizing resource allocation and improving run time.
- `-with-dag` Make a flowchart to show the relationship of channels and processes. 
- [Software dependencies](https://www.nextflow.io/docs/latest/tracing.html#execution-report) to use these features. Note the differences on Mac and Linux.
- How to set them up in the [nextflow.config](https://github.com/AndersenLab/wi-gatk/blob/master/nextflow.config) so they are automatically generated for each run. Credit [danielecook](https://github.com/danielecook) 


### Require users to specify a parameter value
- There are 2 types of parameters: (a) one with no actual value (b) one with actual values. 
- **(a)** If a parameter is specified but no value is given, it is implicitly considered `true`. For example, one can use this to run debug mode `nextflow main.nf --debug`
```
    if (params.debug) {
        ... (set parameters for debug mode)
    } else {
        ... (set parameters for normal use)
    }
```
   - or to print help message `nextflow main.nf --help`
```
    if (params.help) {
        println """
        ... (help msg here)
        """
        exit 0
    }
```

- **(b)** For parameters that need to contain a value, Nextflow recommends to set a default and let users to overwrite it as needed. However, if you want to require it to be specified by the user:
```
    params.reference = null   // no quotes. this line is optional, since without initialising the parameter it will default to null. 
    if (params.reference == null) error "Please specify a reference genome with --reference"
```  

- Below works as long as the user appends a value: `--reference=something`. It will not print the error message with: `nextflow main.nf --reference` (without specifying a value) because this will set `params.reference` to `true` (see point **(a)**) and `!params.reference` will be `false`. 
```
    if (!params.reference) error "Please specify a reference genome with --reference"
```

### Acknowledgement
- [danielecook](https://github.com/danielecook) for offering lots of help and advice.
- The last function `.collect{ it[1] }` in the [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) came from a post in Nextflow Gitter (now replaced by Nextflow Slack) by [Juke34](https://github.com/Juke34)
- [pditommaso](https://github.com/pditommaso) for suggesting edits.
