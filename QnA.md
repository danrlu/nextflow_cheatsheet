What do I do if...

The channel structure is a tuple within tuple, or some formats that neither Nextflow or I can interpret? 

For each item in a channel, the number of sub-items can vary from time to time?

I need to use bash variables in the code block?




### For each item in a channel, the number of sub-items can vary from time to time
A channel has 2 dimensions, each item (or elements as Nextflow calls it) in a row gets its own parallel execution of the process.

<img width="588" alt="image" src="https://user-images.githubusercontent.com/20667188/193308918-7e7ad894-46ec-4668-b5b0-e53a5c48d514.png">

Then within each row, there can be a number of sub-items (not sure the official name) we pass to a process by specifying `input: path(a), path(b)`, which then implicitly requires us to know how many sub-items are there. But we don't always know this ahead of time. For example the number of chromosomes could vary, or the input is sometimes paired-end, sometimes single-end, or some previous processes may not generate an output. The workaround is to put them into a tuple and refer to it as `input: tuple path("*")`.
- This also works when there is tuple within a tuple in the channel 
<img width="1211" alt="image" src="https://user-images.githubusercontent.com/20667188/193298924-0ab9698b-6913-4b63-906a-0618baadf3ff.png">


### I need to use bash variables in the code block
Official doc [here](https://www.nextflow.io/docs/latest/process.html?highlight=bash%20variable#process-shell) and [here](https://www.nextflow.io/docs/latest/process.html?highlight=bash%20variable#process-shell). I recommend the 2nd approach, 
