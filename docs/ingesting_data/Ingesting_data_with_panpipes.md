# Ingesting data with panpipes

Panpipes is a single cell multimodal analysis pipeline with a lot of functionalities to streamline and speed up your single cell projects.

Arguably, the most important part of a pipeline is the ingestion of the data into a format that allows efficient storage and agile processing. We believe that `AnnData` and `MuData` offer all those advantages and that's why we built `panpipes` with these data structure at its core. 
Please check the [`scverse` webpage](https://scverse.org/) for more information on these formats!

We provide examples of how to ingest single cell data from a 10X directory (see the [multiome tutorial](https://panpipes-tutorials.readthedocs.io/en/latest/ingesting_multiome/ingesting_mome.html) or the [citeseq tutorial]()) or directly from existing anndata objects, but we offer readers to load in any tabular format and assay-specific data types into a MuData object (check the [Supported Input Filetypes](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_qc_mm.html#supported-input-filetypes:~:text=per_barcode_metrics_file-,Supported%20input%20filetypes,-%EF%83%81) section and our [Ingesting Spatial Transcriptomics Tutorial](https://panpipes-tutorials.readthedocs.io/en/latest/ingesting_spatial_data/Ingesting_spatialdata_with_panpipes.html))

For all the tutorials we will append the `--local` command which instructs the pipeline to run on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a HPC cluster.


In this tutorial we are starting with the data already in individual h5ad objects per modality. If you want to start from another format, e.g. 10X outputs, or csv matrices, check out the other tutorials and information on supported data formats [here](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_ingest.html)


## Starting from pre-existing h5ad objects

Please download the input data that we have provided [here](https://figshare.com/articles/dataset/data_to_run_tutorials_on_https_github_com_DendrouLab_panpipes-tutorials/23735706). It's a random subset of cells from the [teaseq datasets](https://elifesciences.org/articles/63632) that we also used for the `panpipes` paper.

You should find three `.h5ad` objects in this directory, one for each modality of the teaseq experiment, namely `rna`, `prot` (in this case the object is saved as `adt`) and `atac`.

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

Now move the 3 input anndata you downloaded into the `data.dir` folder you have just created.

```
>ls ingest/data.dir 

adt.h5ad
atac.h5ad
rna.h5ad
```

## Preparing the Config and Submission file for the ingest pipeline

in `teaseq/ingest` call `panpipes ingest config`.
This command will generate a `pipeline.log` and a `pipeline.yml` file. The `pipeline.yml` is our configuration file, it provides the workflow with some essential information, such as the path to the input data, the formats used, and holds the parameters of your analysis. The `ingest` workflow is the only one of the panpipes workflows that requires both a pipeline.yml and a submission file to perform its task (reading in single cell data and formatting it into a MuData object).

This is the sumbission file we are using for this tutorial (we provide it [here](./sample_file_qc.txt)):

| sample_id | rna_path          | rna_filetype | prot_path         | prot_filetype | atac_path          | atac_filetype | tissue |
| --------- | ----------------- | ------------ | ----------------- | ------------- | ------------------ | ------------- | ------ |
| teaseq    | data.dir/rna.h5ad | h5ad         | data.dir/adt.h5ad | h5ad          | data.dir/atac.h5ad | h5ad          | pbmc   |



Let's now take a look at the pipeline.yml file we provide [here](pipeline_yml.md).

```
# ------------------------------------------------------------------------------------------------
# Loading and concatenating data options
# ------------------------------------------------------------------------------------------------
# ------------------------
# Project name and data format
# ------------------------
project: Teaseq
sample_prefix: teaseq
# if you have an existing h5mu object that you want to run through the pipeline then
# store it in the folder where you intend to run the folder, and call it
# ${sample_prefix}_unfilt.h5mu where ${sample_prefix} = sample_prefix argument above
use_existing_h5mu: False

# submission_file format:
# For qc_mm the required columns are
# sample_id  rna_path  rna_filetype  (prot_path  prot_filetype tcr_path  tcr_filetype etc.)
# Example at resources/sample_file_mm.txt
submission_file: sample_file_qc.txt

# which metadata cols from the submission file do you want to include in the anndata object
# as a comma-separated string e.g. batch,disease,sex
metadatacols: 

# which concat join to you want to perform on your mudata objects, recommended inner
# see https://anndata.readthedocs.io/en/latest/concatenation.html#inner-and-outer-joins for details
concat_join_type: inner

#---------------------------------------
# Modalities in the project
#---------------------------------------
# the qc scripts are independent and modalities are processed following this order. Set to True to abilitate modality(ies). 
# Leave empty (None) or False to signal this modality is not in the experiment.

modalities:
  rna: True
  prot: True
  bcr: False
  tcr: False
  atac: True

```
- We specify the name of the sample that will be generated by aggregating the multimodal single cell data into a single MuData
- We provide the pipeline with the input submission file in the current directory
- We instruct the pipeline to perform the inner joint of loaded the cells in each unimodal assay (that is, we are working on cells which have all the three modalities and discarding those that don't have measurements in less than the 3 specified)

other parameters include:
- running doublet detection on RNA using scrublet
```
scr:
  run: True
```
- normalization choice of protein data

```
normalisation_methods: clr
# CLR parameters:
# margin determines whether you normalise per cell (as you would RNA norm), 
# or by feature (recommended, due to the variable nature of adts). 
# CLR margin 0 is recommended for informative qc plots in this pipeline
# 0 = normalise colwise (per feature)
# 1 = normalise rowwise (per cell)
clr_margin: 1
```

- qc covariates to plot for each modality.

```
# ------------
# Plot RNA QC metrics
# ------------
# all metrics should be inputted as a comma separated string e.g. a,b,c
# base of the plots, normally is the channel also referred to "sample_id"
plotqc_grouping_var: orig.ident
# other cell covariates
plotqc_rna_metrics: doublet_scores,pct_counts_mt,pct_counts_rp,pct_counts_hb,pct_counts_ig

# ------------
# Plot PROT QC metrics
# ------------
# requires prot_path to be included in the submission file
# all metrics should be inputted as a comma separated string e.g. a,b,c

# as standard the following metrics are calculated for prot data
# per cell metrics:
# total_counts,log1p_total_counts,n_adt_by_counts,log1p_n_adt_by_counts
# if isotypes can be detected then the following are calculated also:
# total_counts_isotype,pct_counts_isotype
# choose which ones you want to plot here
plotqc_prot_metrics: total_counts,log1p_total_counts,n_adt_by_counts,pct_counts_isotype
# Since the protein antibody panels usually count fewer features than the RNA, it may be interesting to
# visualize breakdowns of single proteins plots describing their count distribution and other qc options.
# choose which ones you want to plot here, for example
# n_cells_by_counts,mean_counts,log1p_mean_counts,pct_dropout_by_counts,total_counts,log1p_total_counts
prot_metrics_per_prot: total_counts,log1p_total_counts,n_cells_by_counts,mean_counts


```

Inspect the configuration `pipeline.yml` file to familiarize with all the options.
You can modify the `pipeline.yml` with custom parameters, or simply replace with the one we provide [here](pipeline_yml.md).
Please note that the parameters specified for modalities that are not part of the experiment are ignored by the worlfow (in this case we are not processing bcr and tcr).


### Specifying paths

As part of the ingest workflow, we additionally specify a series of custom paths to the files that contains the genes used for qc'ing the cells. We provide an example file that contains gene pathways that are associated to commonly used signatures, like mitochondrial or ribosomal genes.
These genes are used to score the cells for enrichment of specific signatures, and to flag cells with high percentage mitochondrial reads.

Download this file [here](./qc_genelist_1.0.csv)

remember to change the following paths to the files on your local machine! 

- We need to provide the correct conda environment if using one (you can also leave empty if using python venv)
- We need to specify the path to the `sample_qc_file.txt` submission file 
- We need to provide the path to the `qc_genelist_1.0.csv` file that you downloaded or customized.

## Running the workflow

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









