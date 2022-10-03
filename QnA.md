What to do if...


## What to do if the number of items in the input channel is unknown
Each input channel has 2 dimensions: how many items (each item will have its own separate execution of the process), and within each item how many sub-items (not sure the official name for these). This section is about the latter case. In process definition, we needs to specify `input: val(a), path(b)`, which implicitly requires us to know the number of sub-items in the channel. However, the number of sub-items can vary, for example, if we want to combine all fastq files in a folder, or the number of chromosomes vary from species to species, in which case, we can use
```
input: tuple path("*")
```
The above will create a symlink in the working directory for each of the files in the input channel, so the script/code section can treat them as if these files exist in the current folder. 

A more complicated case from the [cheatsheet](https://github.com/danrlu/nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf): 
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
