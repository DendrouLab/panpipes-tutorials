# Ingesting data with panpipes

Panpipes is a single cell multimodal analysis pipeline with a lot of functionalities to streamline and speed up your single cell projects.

Arguably, the most important part of a pipeline is the ingestion of the data into a format that allows efficient storage and agile processing. We believe that `AnnData` and `MuData` offer all those advantages and that's why we built `panpipes` with these data structure at its core. 
Please check the [`scverse` webpage](https://scverse.org/) for more information!

We will give you a couple of examples of reading data from a 10X directory or directly from existing anndata objects, but we offer functionalities to read in any tabular format and assay-specific data types (check out Spatial Transcriptomics and Repertoire analysis)

For all the tutorials we will prepend the `--local` command which ensures that the pipeline runs on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a cluster.


In this tutorial we are starting with the data already in individual h5ad objects per modality. If you want to start from another format, e.g. 10X outputs, or csv matrices, there is lots of information [here](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_ingest.html)


## Starting from pre-existing h5ad objects

Please download the input data that we have provided [here](https://figshare.com/articles/dataset/data_to_run_tutorials_on_https_github_com_DendrouLab_panpipes-tutorials/23735706). It's a random subset of cells from the [teaseq datasets](https://elifesciences.org/articles/63632) that we also used for the `panpipes` paper.

You should find three `.h5ad` objects in this directory, one for each modality of the teaseq experiment, namely `rna`, `adt` and `atac`.

In order to ingest the data, we have to tell panpipes the paths to each anndata.

Download an example sample submission file here: [sample_file_qc.txt](sample_file_qc.txt).


Create a directory in which you will store all the processing steps.
for example 

``` 
mkdir teaseq
```


This is the top level directory in which you will create individual workflows outputs. 
Create the directory to run the `ingest` workflow.

```
cd teaseq
mkdir ingest && cd ingest

mkdir data.dir
```

Now move the 3 anndata you downloaded to the `data.dir` folder you have just created.

in `teaseq/ingest` call `panpipes ingest config`.
this will generate a `pipeline.log` and a `pipeline.yml` file.


Modify the `pipeline.yml` with custom parameters: you can simply replace with the one we provide [here](pipeline_yml.md).
We specify a series of custom paths such as the conda environment or the path to the file that contains the genes used for qc'ing the cells. We provide an example file that contains gene pathways that are associated to commonly used signatures, like mitochondrial or ribosomal genes.
These genes are used to score the cells for enrichment of specific signatures, and to flag cells with high percentage mitochondrial reads.

Download this file from [here](./qc_genelist_1.0.csv)

Inspect the `pipeline.yml` file to familiarize with the input parameters. 

NOTE: remember to change the following paths to the files on your local machine! 

- We need to provide the correct conda environment if using one (you can also leave empty if using python venv)
- We need to specify the path to the `sample_qc_file.txt` file 
- We need to provide the path to the `qc_genelist_1.0.csv` file that you downloaded.




Review the steps that are going to be run as part of the `ingest` pipeline:
```
panpipes ingest show full --local
``` 


This is the output which describes all the tasks which will be run:
```
Task = "mkdir('logs') #2   before pipeline_ingest.load_mudatas "
Task = 'pipeline_ingest.load_mudatas'
Task = 'pipeline_ingest.concat_filtered_mudatas'
Task = 'pipeline_ingest.run_scrublet'
Task = 'pipeline_ingest.run_rna_qc'
Task = "mkdir('logs') #2   before pipeline_ingest.load_bg_mudatas "
Task = 'pipeline_ingest.load_bg_mudatas'
Task = 'pipeline_ingest.downsample_bg_mudatas'
Task = 'pipeline_ingest.concat_bg_mudatas'
Task = 'pipeline_ingest.run_scanpy_prot_qc'
Task = 'pipeline_ingest.run_dsb_clr'
Task = 'pipeline_ingest.run_prot_qc'
Task = 'pipeline_ingest.run_repertoire_qc'
Task = 'pipeline_ingest.run_atac_qc'
Task = 'pipeline_ingest.run_qc'
Task = 'pipeline_ingest.run_assess_background'
Task = 'pipeline_ingest.plot_qc'
Task = 'pipeline_ingest.all_rna_qc'
Task = 'pipeline_ingest.all_prot_qc'
Task = "mkdir('logs')   before pipeline_ingest.aggregate_tenx_metrics_multi "
Task = 'pipeline_ingest.aggregate_tenx_metrics_multi'
Task = 'pipeline_ingest.process_all_tenx_metrics'
Task = 'pipeline_ingest.full'
```


To run the ingest complete workflow 

```
panpipes ingest make full --local
``` 

`panpipes ingest` will produce a different files including tab-separated metadata, plots and most notably a `*_unfilt.h5mu` object containing all the cells and the metadata, with calculated QC metrics such as `pct_counts_mt`,`pct_counts_mt` for percentage mitochondrial/ribosomal reads, `total_counts` for any of the supported modalities that use this info (such as rna, prot or atac), and other custom ones you may speficy by customizing the `pipeline.yml`.

You can also run individual steps, i.e. `panpipes ingest make plot_qc --local` will produce the qc plots from the metadata you have generated. In the `pipeline_ingest.py` workflow script, you can see that this step follows the qc metrics calculation for the multimodal assays.


```
@follows(run_rna_qc, run_prot_qc, run_repertoire_qc, run_atac_qc)
def run_qc():
    pass
```
So the pipeline will try to pick up from there, or produce the qc outputs that are missing in order to have the inputs for this task.
If you have run a full workflow and want to change the parameters in the yaml and reproduce the output of one task, you will have to remove the log file for that task so the pipeline knows where to start from!  

## Next Steps: 

Filtering the cells using [panpipes preprocess](../filtering_data/filtering_data_with_panpipes.md)




*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*









