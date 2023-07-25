## Ingesting data with panpipes

Panpipes is a single cell multimodal analysis pipeline with a lot of functionalities to streamline and speed up your single cell projects.

Arguably, the most important part of a pipeline is the ingestion of the data into a format that allows efficient storage and agile processing. We believe that `AnnData` and `MuData` offer all those advantages and that's why we built `panpipes` with these data structure at its core. 
Please check the [`scverse` webpage](https://scverse.org/) for more information!

We will give you a couple of examples of reading data from a 10X directory or directly from existing anndata objects, but we offer functionalities to read in any tabular format and assay-specific data types (check out Spatial Transcriptomics and Repertoire analysis)

For all the tutorials we will prepend the `--local` command which ensures that the pipeline runs on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a cluster.

### Start from 10X directories

lorem ipsum

### Start from pre-existing h5ad objects

Please download the input data that we have provided [here](https://figshare.com/articles/dataset/data_to_run_tutorials_on_https_github_com_DendrouLab_panpipes-tutorials/23735706). It's a random subset of cells from the [teaseq datasets](https://elifesciences.org/articles/63632) that we also used for the `panpipes` paper.

You should find three `.h5ad` objects in this directory, one for each modality of the teaseq experiment, namely `rna`, `adt` and `atac`.

In order to ingest the data, we have to tell panpipes the paths to each anndata.
Create a csv file like the one we provide in the [tutorials](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/ingesting_data), if you have cloned this repo, you should have it under `sample_file_qc.txt`.

create a directory in which you will store all the processing steps.
for example 

``` 
mkdir teaseq
```

this is the upper level directory in which you will create individual workflows outputs. 
Create the directory to run the `qc_mm` (aka ingestion) workflow.

```
cd teaseq
mkdir qc_mm && cd $_
mkdir data.dir
```

Now move the 3 anndata you downloaded to the `data.dir` folder you have just created.

in `teaseq/qc_mm` call `panpipes qc_mm config`.
this will generate a `pipeline.log` and a `pipeline.yml` file.

Modify the `pipeline.yml` with custom parameters or simply replace with the one we provide in [tutorials](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/ingesting_data)

type `panpipes qc_mm show full --local` to see what will be run.

```
Task = "mkdir('logs') #2   before pipeline_qc_mm.load_mudatas "
Task = 'pipeline_qc_mm.load_mudatas'
Task = 'pipeline_qc_mm.concat_filtered_mudatas'
Task = 'pipeline_qc_mm.run_scrublet'
Task = 'pipeline_qc_mm.run_rna_qc'
Task = "mkdir('logs') #2   before pipeline_qc_mm.load_bg_mudatas "
Task = 'pipeline_qc_mm.load_bg_mudatas'
Task = 'pipeline_qc_mm.downsample_bg_mudatas'
Task = 'pipeline_qc_mm.concat_bg_mudatas'
Task = 'pipeline_qc_mm.run_scanpy_prot_qc'
Task = 'pipeline_qc_mm.run_dsb_clr'
Task = 'pipeline_qc_mm.run_prot_qc'
Task = 'pipeline_qc_mm.run_repertoire_qc'
Task = 'pipeline_qc_mm.run_atac_qc'
Task = 'pipeline_qc_mm.run_qc'
Task = 'pipeline_qc_mm.run_assess_background'
Task = 'pipeline_qc_mm.plot_qc'
Task = 'pipeline_qc_mm.all_rna_qc'
Task = 'pipeline_qc_mm.all_prot_qc'
Task = "mkdir('logs')   before pipeline_qc_mm.aggregate_tenx_metrics_multi "
Task = 'pipeline_qc_mm.aggregate_tenx_metrics_multi'
Task = 'pipeline_qc_mm.process_all_tenx_metrics'
Task = 'pipeline_qc_mm.full'
```

Now run the qc_mm complete workflow 

`panpipes qc_mm make full --local` 

`panpipes qc_mm` will produce a different files including tab-separated metadata, plots and most notably a `*_unfilt.h5mu` object containing all the cells and the metadata, with calculated QC metrics such as `pct_counts_mt`,`pct_counts_mt` for percentage mitochondrial/ribosomal reads, `total_counts` for any of the supported modalities that use this info (such as rna, prot or atac), and other custom ones you may speficy by customizing the `pipeline.yml`.

You can also run individual steps, i.e. `panpipes qc_mm make plot_qc --local` will produce the qc plots from the metadata you have generated. In the `pipeline_qc_mm.py` worflow script, you can see that this step follows the qc metrics calculation for the multimodal assays.

```
@follows(run_rna_qc, run_prot_qc, run_repertoire_qc, run_atac_qc)
def run_qc():
    pass
```
So the pipeline will try to pick up from there, or produce the qc outputs that are missing in order to have the inputs for this task.
If you have run a full workflow and want to change the parameters in the yaml and reproduce the output of one task, you will have to remove the log file for that task so the pipeline knows where to start from!  

#### [Next: filtering the cells using `panpipes preprocess`](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/filtering_data/filtering_data_with_panpipes.md)













