What to do if...


## What to do if the number of items in the input channel is unknown
Each input channel has 2 dimensions: how many items (each item will have its own separate execution of the process), and within each item how many sub-items (not sure the official name for these). This section is about the latter case. In process definition, one needs to specify `input: val(a), path(b)`, which implicitly requires us to know the number of sub-items in the channel. However, the number of sub-items can vary, for example, if we want to combine all fastq files in a folder, or the number of chromosomes vary from species to species, in which case, we can use
```
input: tuple path("*")
```
A more complicated case: 
<img width="1194" alt="image" src="https://user-images.githubusercontent.com/20667188/193614589-46771b87-ec37-4b0d-825f-8028550704ae.png">
