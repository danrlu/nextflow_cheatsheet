# For more complicated workflows

### [Official Nextflow patterns](https://nextflow-io.github.io/patterns/index.html)

### Places to search for answers:
- [Nextflow patterns, official](https://nextflow-io.github.io/patterns/index.html)
- [Gitter chat room](https://gitter.im/nextflow-io/nextflow)
- [Google group](https://groups.google.com/forum/#!forum/nextflow)
- The old posts in these places are a treasure dump that answered 99% of my questions. As an example, the last function `.collect{ it[1] }` in the [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) came from a post in Gitter by [@Juke34](https://github.com/Juke34)

### Require users to sepcify a parameter value
- There are 2 types of paramters: (a) one with no actual value (b) one with actual values. 
- **(a)** If a parameter is specified but no value is given, it is implicitly considered `true`. So one can use this to run debug mode `nextflow main.nf --debug`
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

- Below works as long as the user always append a value: `--reference=something`. It will not print the error message with: `nextflow main.nf --reference` (without specifying a value) because this will set `params.reference` to `true` (see point **(a)**) and `!params.reference` will be `false`. 
```
    if (!params.reference) error "Please specify a reference genome with --reference"
```
