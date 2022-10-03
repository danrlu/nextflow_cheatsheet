What to do if...


## What to do if the number of items in the input channel is unknown or can vary from time to time
Each input channel has 2 dimensions: each item (or elements as Nextflow calls it) in a row gets its own parallel execution of the process. 
<img width="494" alt="image" src="https://user-images.githubusercontent.com/20667188/193625668-151afb1e-1248-422f-8670-8802d9abd928.png">

Then within each row, there can be a number of sub-items (not sure the official name) we pass to a process by specifying `input: path(a), path(b)`, which then implicitly requires us to know how many sub-items are there. But we don't always know this ahead of time. For example, if the input is all fastq files in a folder, or the number of chromosomes vary from species to species. The workaround is:
```
input: path("*")
```
The above will create a symlink in the working directory for each of the files in the input channel, so the script/code section can treat them as if these files exist in the current folder. 

A more complicated case from the [cheatsheet](https://github.com/danrlu/nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) to handle additional input files: 
<img width="1194" alt="image" src="https://user-images.githubusercontent.com/20667188/193614589-46771b87-ec37-4b0d-825f-8028550704ae.png">


## What to do if I want to use bash variables in the code block
Often we use `"""` to indicate the code section and `$a` (which is a Nextflow variable) to refer to items passed from the input channel. Then if we want to use a bash variable in the code section, it needs to be escaped `\$b`. Alternatively, which is more readable and less confusing, we can specify a code block to be a [shell block](https://www.nextflow.io/docs/latest/process.html?highlight=debug#process-shell), and bash variables can be treated as normally in bash, and input channel items can be referred to using `!`
```
shell:
'''
echo !{nextflow_variable}
echo ${bash_variable}
'''
```
