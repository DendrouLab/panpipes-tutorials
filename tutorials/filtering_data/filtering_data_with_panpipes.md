## Filtering data with panpipes

You have ingested your single cell data and you have created a `.h5mu` object storing calculated qc metrics.
It's time to filter the cells to make sure to exclude bad quality cells for downstream analysis.

go to your previously created `teaseq` directory and create a new folder to run `panpipes preprocess`

```
# if you are in teaseq/qc_mm
# cd ..
mkdir preprocessing & cd $_
```

In here, run `panpipes preprocess config --local`, which will generate again a `pipeline.yml` file for you to customize. We provide this yml in the [filtering_data](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/filtering_data) folder in this repo.

Open the yml file to inspect the parameters choice. 

If you have run the previous step, [Ingesting data with panpipes]() you will have a "*.unfilt.h5mu" object that you want to apply filtering on. You may also have a custom `.h5mu` dataset that you didn't generate with the `panpipes qc_mm` workflow, but you can use `panpipes preprocess` workflow to filter the cells according to your own qc criteria. 
Either copy the file into the directory you have just created or link the file into the new directory (our preferred choice)

```
cd preprocessing
ln -s ../teaseq_unfilt.h5mu .
# equivalent of
ln -s ../teaseq_unfilt.h5mu teaseq_unfilt.h5mu 
```

**NOTE**: it's important that the `.h5mu` object is linked (or renamed) in this directory with a `sampleprefix_unfilt` structure, cause the pipeline will look for this string to start from.

In the yaml, we have already specified that this is the input file

```
# Start
# --------------------------

sample_prefix: teaseq
unfiltered_obj: teaseq_unfilt.h5mu
# if running this on prefiltered datat then
#1. set unfiltered obj (above) to blank
#2. rename your filtered file to match, the format PARAMS['sample_prefix'] + '.h5mu'
#3. put renamed file in the same folder as this yml.
#4. set filtering run: to False below.
```

now run `panpipes preprocess make full --local`

Look at the log file as it's generated `logs/filtering.log` to see what percentage of cells you are filtering based on the selection criteria we specified in the yml.

As usual, you can choose to modify the parameters and re-run a specific task, for example `panpipes preprocess make rna_preprocess` will specifically run the filtering task on the rna and exit the pipeline.

#### Next [uni and multimodal integration](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/uni_multi_integration/Integrating_data_with_panpipes.md)




