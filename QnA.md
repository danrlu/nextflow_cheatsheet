What to do if...
[the number of items in the input channel is unknown](#the-number-of-items-in-the-input-channel-is-unknown)

## the number of items in the input channel is unknown
Each input channel has 2 dimensions: each item (or elements as Nextflow calls it) in a row gets its own parallel execution of the process. 
<img width="494" alt="image" src="https://user-images.githubusercontent.com/20667188/193625668-151afb1e-1248-422f-8670-8802d9abd928.png">

Then within each row, there can be a number of sub-items (not sure the official name). This section is about the latter case. In the process definition,  we need to specify inputs with `input: path(a), path(b)`, which then implicitly requires us to know how many sub-items are there. But we don't always know this ahead of time. For example, if the input is all fastq files in a folder, or the number of chromosomes vary from species to species, etc.. The workaround is:
```
input: path("*")
```
The above will create a symlink in the working directory for each of the files in the input channel, so the script/code section can treat them as if these files exist in the current folder. 

A more complicated case from the [cheatsheet](https://github.com/danrlu/nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) to handle additional input files: 
<img width="1194" alt="image" src="https://user-images.githubusercontent.com/20667188/193614589-46771b87-ec37-4b0d-825f-8028550704ae.png">


## I want to use bash variables in the code block
Often we use `"""` to indicate the code section and `$a` (which is a Nextflow variable) to refer to items passed from the input channel. Then if we want to use a bash variable in the code section, it needs to be escaped `\$b`. Alternatively, which is more readable and less confusing, we can specify a code block to be a [shell block](https://www.nextflow.io/docs/latest/process.html?highlight=debug#process-shell), and bash variables can be treated as normally in bash, and input channel items can be referred to using `!`
```
shell:
'''
echo !{nextflow_variable}
echo ${bash_variable}
'''
```


## To weave in R scripts
I'm a big fan of using Rmarkdown to generate a html report of the stats/results. There are a few pieces to solve to get this to work in a Nextflow pipeline:
- Have R and its packages in the environment. This is a project of its own. tl;dr is it is best is to have a conda environment dedicated for R and pass it to Nextflow. But occassionally a package cannot be installed through conda, then it's good to set up R without conda entirely. If you mix R inside of conda with R outside of conda, sometimes the library path needs manual adjustment and that gets complicated. Read more [here](https://community.rstudio.com/t/why-not-r-via-conda/9438/4) and [here](https://www.biostars.org/p/450316/).
- The way Rmarkdown work is you write code chunks in a .Rmd file, then knit it and it will by default read in input files in the same folder as the .Rmd file. With Nextflow there are 2 things to keep track of:
+ Where is the .Rmd file. Remember when you do `Channel.fromPath("/path/my.Rmd")`, Nextflow puts a symlink into the working directory, and when knitting, the link points back to where the .Rmd file originally is at `/path/my.Rmd`!
+ Are the input files in the same folder as the .Rmd file
```
    publishDir "${params.out}/species_check/", mode: 'copy', pattern: '*.html'
    
    input:
        tuple path(a), path(rmd_template)

    output:
        tuple path("*.tsv"), path("*.Rmd"), path("*.html")

    """
    # pass variables to the R script
    Rscript --vanilla ${workflow.projectDir}/bin/code.R ${params.fastq_folder} ${a}

    # update code in the ${rmd_template} which is the template .Rmd file if needed
    # here FQ_HOLDER is meant to be updated for each run using fastqa_folder
    cat "${report}" | sed "s/FQ_HOLDER/${params.fastq_folder}/g" > out_${params.fastq_folder}.Rmd 

    # knit the .Rmd file. 
    # this will look for input files in the current working directory. If the input files are not generated in this code block, they need to come from input channels so they exist in the current working directory
    Rscript -e "rmarkdown::render('out_${params.fastq_folder}.Rmd')"
    
    # alternatively, if we want to knit the .Rmd in another directory
    rmarkdown::render('out_${params.fastq_folder}.Rmd', knit_root_dir = "folder/" )
    """
```
