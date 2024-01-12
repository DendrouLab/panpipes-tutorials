# Ingesting data with panpipes

Panpipes is a single cell multimodal analysis pipeline with a lot of functionalities to streamline and speed up your single cell projects.

Arguably, the most important part of a pipeline is the ingestion of the data into a format that allows efficient storage and agile processing. We believe that `AnnData` and `MuData` offer all those advantages and that's why we built `panpipes` with these data structure at its core. 
Please check the [`scverse`](https://scverse.org/) webpage for more information on these formats!

When running panpipes, you always need to start with the data ingestion. The ingest workflow will do the following:
- create summary plots of 10x metrics
- calculate scrublet scores (doublet detection)
- scanpy Quality Control (QC)
- create summary QC plots

The ingest pipeline does not perform any filtering of cells or genes. Filtering occurs as the first step in the [preprocess workflow](https://panpipes-tutorials.readthedocs.io/en/latest/filtering_data/filtering_data_with_panpipes.html), which should be executed after the ingestion workflow.

We provide examples of how to ingest single cell data from a 10X directory (see the [multiome tutorial](https://panpipes-tutorials.readthedocs.io/en/latest/ingesting_multiome/ingesting_mome.html) or the [citeseq tutorial]()) or directly from existing anndata objects, but we offer the possibility load any tabular format and assay-specific data types into a MuData object (check the [Supported Input Filetypes](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_qc_mm.html#supported-input-filetypes:~:text=per_barcode_metrics_file-,Supported%20input%20filetypes,-%EF%83%81) section and our [Ingesting Spatial Transcriptomics Tutorial](https://panpipes-tutorials.readthedocs.io/en/latest/ingesting_spatial_data/Ingesting_spatialdata_with_panpipes.html))

For all the tutorials we will append the `--local` command which instructs the pipeline to run on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a HPC cluster.

Note that if you are combining multiple datasets from different sources, the final `AnnData` object will only contain the intersection of the genes from all the datasets. For example, if the mitochondrial genes have been excluded from one of the inputs, they will be excluded from the final dataset. In this case, it might be wise to run ingest separately on each dataset and then merge them together to create one `AnnData` object to use as input for the integration workflow.

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
- normalization choice of protein data. Since we don't have access to cellranger files to estimate the background protein data, the only normalization we can use is clr

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
These genes are used to score the cells for enrichment of specific signatures, and to flag cells with high percentage mitochondrial reads. We explain more on how to supply and use these custom genes list in the [usage section](https://panpipes-pipelines.readthedocs.io/en/latest/usage/gene_list_format.html), check it out! 

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

Now to run the complete ingest workflow type:

```
panpipes ingest make full --local
``` 
That's it. If running successfully you should see that the pipeline prints the tasks it's running to the stdout.


This output is also appended to the `pipeline.log` file that was generated when configuring the workflow, so you can keep track of what is happening.

```
# 2023-11-20 14:37:45,205 INFO running statement: \
#                              python /Users/fabiola.curion/Documents/devel/github/panpipes/panpipes/python_scripts/make_adata_from_csv.py          --mode_dictionary "{'rna': True, 'prot': True, 'bcr': False, 'tcr': False, 'atac': True}"         --sample_id teaseq         --output_file ./tmp/teaseq.h5mu       --use_muon True --rna_infile data.dir/rna.h5ad --rna_filetype h5ad --prot_infile data.dir/adt.h5ad --prot_filetype h5ad --subset_prot_barcodes_to_rna False --atac_infile data.dir/atac.h5ad --atac_filetype h5ad > logs/load_mudatas_teaseq.log

# 2023-11-20 14:38:36,863 INFO {"task": "pipeline_ingest.load_mudatas", "engine": "LocalExecutor", "statement": "python /Users/fabiola.curion/Documents/devel/github/panpipes/panpipes/python_scripts/make_adata_from_csv.py          --mode_dictionary \"{'rna': True, 'prot': True, 'bcr': False, 'tcr': False, 'atac': True}\"         --sample_id teaseq         --output_file ./tmp/teaseq.h5mu       --use_muon True --rna_infile data.dir/rna.h5ad --rna_filetype h5ad --prot_infile data.dir/adt.h5ad --prot_filetype h5ad --subset_prot_barcodes_to_rna False --atac_infile data.dir/atac.h5ad --atac_filetype h5ad > logs/load_mudatas_teaseq.log", "job_id": 71009, "slots": 1, "start_time": 1700487466.1516511, "end_time": 1700487516.860606, "submission_time": 1700487466.1516511, "hostname": "MB084405", "total_t": 50.70895481109619, "exit_status": 0, "user_t": 44.91, "sys_t": 2.33, "wall_t": 50.67, "shared_data": 0, "io_input": 0, "io_output": 0, "average_memory_total": 0, "percent_cpu": 93.15908832272535, "average_rss": 0, "max_rss": 712700, "max_vmem": 712700, "minor_page_faults": 285052, "swapped": 0, "context_switches_involuntarily": 22936, "context_switches_voluntarily": 6284, "average_uss": 0, "signal": 1, "major_page_fault": 3860, "unshared_data": 0, "cpu_t": 47.239999999999995}

# 2023-11-20 14:38:36,869 INFO {"task": "'pipeline_ingest.load_mudatas'", "task_status": "completed", "task_total": 1, "task_completed": 0, "task_completed_percent": 0.0}
# 2023-11-20 14:38:36,869 INFO Completed Task = 'pipeline_ingest.load_mudatas' 

```


`panpipes ingest` will produce a host of different files including tab-separated metadata, plots and most notably a `*_unfilt.h5mu` object containing all the cells and the metadata, with calculated QC metrics such as `pct_counts_mt`,`pct_counts_mt` for percentage mitochondrial/ribosomal reads, `total_counts` for any of the supported modalities that use this info (such as rna, prot or atac), and other custom ones you may speficy by customizing the `pipeline.yml`.


| file                         | type file | info                                                                                                                                                         |
| ----------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| data.dir                            | directory | the folder with input files we organized                                                                                                                     |
| figures                             | directory | folder storing plots generated throughout the ingest workflow                                                                                                |
| logs                                | directory | folder storing logs generated throughout the ingest workflow                                                                                                 |
| teaseq_cell_metadata.tsv              | text file | the cell metadata of the experiment (the .obs slot of the mudata object) saved as a tsv file                                                                 |
| teaseq_threshold_filter.tsv           | text file | summary file showing percentage of cells remaining for commonly used QC thresholds on genes and cells. This filtering is not applied in the ingest workflow. |
| teaseq_threshold_filter_explained.tsv | text file | file containing the thresholds used to produce the previous file                                                                                             |
| teaseq_unfilt.h5mu                    | h5mu      | the mudata generated from the input files                                                                                                                    |
| sample_file_qc.txt                     | text file | input sample submission file                                                                                                                                 |
| pipeline.log                        | log       | pipeline log file, stores all the info on the commands run                                                                                                   |
| pipeline.yml                        | yaml      | input yaml file                                                                                                                                              |
| scrublet                            | directory | directory storing the scrublet analysis results                                                                                                              |
| tmp                                 | directory | directory storing temporary h5mu files (one for each sample in the submission file)                                                                          |
|                                     |           |                                                                                                                                                              |

Some example figures include: 
- Scatterplots and violin plots for each modality 

In this example, a violin plot of the computed atac metrics

<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/ingesting_data/figures/atac/violinatac_metrics_violin.png?raw=true" alt="img1" >


Or a scatterplot of the RNA total counts vs percentage genes mapping to mitochondrial reads in each cell.
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/ingesting_data/figures/rna/scatter_sample_id_rna-nUMI_v_rna-pct_mt.png?raw=true" alt="img1" >


- Ridgeplots of CLR normalized protein values in the experiment

<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/ingesting_data/figures/prot/teaseq_clr_ridgeplot.png?raw=true" alt="img1" height=100 width =2000>
	

Particularly interesting may be plots comparing one modality to the other, for example exploration of normalized PROT vs RNA counts can provide a quick overview of cells where one modality may be incomplete or sparser than expected (see the cells that strongly deviate from the expected correlation)

<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/ingesting_data/figures/rna_v_prot/scatter_atac.orig.ident-log1p_nUMI_v_rna-log1p_nUMI.png?raw=true" alt="img1" >

The same plot can be produced for different grouping covariates. In this example we used the "sample_id" which is the value we specified in the submission file, that will become the sample name of the concatenated MuData object produced after the ingestion.

Since the the unimodal `annadata` we used for the tutorial also had the original samples covariates stored in the `.obs`,  we have used the covariate "orig.ident" from the protein modality to show the scatterplot of log1p_nUMI in RNA vs PROT across the 3 teaseq samples of the original experiment.

<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/ingesting_data/figures/rna_v_prot/scatter_prot.orig.ident-log1p_nUMI_v_rna-log1p_nUMI.png?raw=true" alt="img1" >

the naming of the plots allows to identify the output:

scatter_prot.orig.ident-log1p_nUMI_v_rna-log1p_nUMI.png:

- **scatter**: the type of plot (values: scatter/violin/barplot)
  
- **prot.orig.ident**: the grouping var used ("orig.ident" from the prot modality)
  
- **log1p_nUMI_v_rna-log1p_nUMI**: the X and Y coordinates of the scatterplot


You can also run individual steps, i.e. `panpipes ingest make plot_qc --local` will produce the qc plots from the metadata you have generated. In the `pipeline_ingest.py` workflow script, you can see that this step follows the qc metrics calculation for the each of the multimodal assays.


```
@follows(run_rna_qc, run_prot_qc, run_repertoire_qc, run_atac_qc)
def run_qc():
    pass

@follows(run_qc)
@originate("logs/plot_qc.log", orfile())
def plot_qc(log_file, cell_file):
    qcmetrics = PARAMS['plotqc_rna_metrics']
    cmd = """
    Rscript %(r_path)s/plotQC.R 
    --prefilter TRUE
    --cell_metadata %(cell_file)s 
    --sampleprefix %(sample_prefix)s
    --groupingvar %(plotqc_grouping_var)s
[...]
```

If you have run a full workflow and want to reproduce the output of one task (for example you have changed some of the parameters in the yml file), you have to remove the outputs of that task from the directory, so the pipeline knows where to start from.  
To reproduce the plots, you have to remove the `plot_qc.log` file from the logs folder, and the workflow will pick up from there.
Please note that there are more complex tasks involving multiple output files, so in order to reproduce a step or the entire workflow for a different set of parameters it's advisable to remove all outputs.

## Next Steps: 

Filtering the cells using [panpipes preprocess](../filtering_data/filtering_data_with_panpipes.md)




*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*









