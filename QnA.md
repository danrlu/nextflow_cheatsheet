Start here: official Nextflow patterns

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
I'm a big fan of using Rmarkdown to generate a html report to plot the results. There are many ways to get this to work but below is what I usually do:
- Have R and its packages in the environment. This is a project of its own. tl;dr is it is best is to have a conda environment dedicated for R and pass it to Nextflow. But occassionally a package cannot be installed through conda, then it's good to set up R without conda entirely. If you mix R inside of conda with R outside of conda, sometimes the library path needs manual adjustment and that gets complicated. Read more [here](https://community.rstudio.com/t/why-not-r-via-conda/9438/4) and [here](https://www.biostars.org/p/450316/).
- The way Rmarkdown work is you write code chunks in a .Rmd file, then knit it and it will by default read in input files in the same folder as the .Rmd file. In the context of Nextflow, there are 2 things to keep track of:
    + Where is the .Rmd file. Remember when we do `Channel.fromPath("/path/my.Rmd")`, Nextflow by default puts a symlink into the working directory (note the size below), and when knitting, the file that the symlink points to get knitted in its original folder, which is `/path/my.Rmd` instead of the working directory. 
    + <img width="445" alt="image" src="https://user-images.githubusercontent.com/20667188/193975974-2ce12deb-21db-45c2-84b1-69e6a94b75f3.png">
    + The input files or their symlinks need to be in the same folder as the .Rmd file. So the easiest way:
```
process generate_html {

    conda "/Users/dlu/miniconda3/envs/r"
    publishDir "${params.out}", mode: 'copy', pattern: '*'
    
    input:
        tuple path(a), path(rmd_template)

    output:
        tuple path("*.Rmd"), path("*.html")

    """
    # create an actual copy of the .Rmd code in the working directory
    # note ${rmd_template} here is a symlink
    cat "${rmd_template}" > plot.Rmd 
    
    # OR if needed, update code in the ${rmd_template} 
    # here FQ_HOLDER is meant to be replaced/updated for each run using params.fastqa_folder
    # cat "${rmd_template}" | sed "s/FQ_HOLDER/${params.fastq_folder}/g" > plot.Rmd 

    # knit the .Rmd file. 
    # this will look for input files in the current working directory. in this example, the input file is passed to the process via path(a)
    Rscript -e "rmarkdown::render('plot.Rmd')"
    """
}
```
- Another approach is to knit it in the output directory if all inputs for the .Rmd are already written out there. The benefit here is I can modify the .Rmd and re-knit it easily after Nextflow runs.
```
process generate_html {

    conda "/Users/dlu/miniconda3/envs/r"
 
    input:
        tuple path(a), path(rmd_template) // Even though path(a) is not used by the .Rmd in the code block below, it serves as the signal when this process can start to run. 

    // there will be nothing to output or publish since all files will already be in the desired output directory    

    """
    # create an actual copy of the .Rmd code in the working directory
    # note ${rmd_template} here is a symlink
    cat "${rmd_template}" > ${params.out}/plot.Rmd 
    
    # here we're leaving the working directory  
    # all output files will no longer be in the working directory and can no longer be found by Nextflow to put into output channels
    cd ${params.out} 
    
    # knit the .Rmd file
    # the resulting .html file will be in the same folder, aka ${params.out}
    Rscript -e "rmarkdown::render('plot.Rmd')"
    """
}
```
