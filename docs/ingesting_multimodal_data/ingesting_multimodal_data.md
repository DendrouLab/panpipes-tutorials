Ingesting multimodal (CITE-Seq + VDJ) datasets from cellranger outputs 
===================

## Datasets and directories
Panpipes can read files from different cellranger outputs. Here we will showcase the example of  two multmodal (CITE-Seq + VDJ) datasets from the 10x website while ingesting cellranger multi outputs. 

Download **dataset 1 ( 10k human PBMCs)** from the [10x website](https://www.10xgenomics.com/resources/datasets/integrated-gex-totalseq-c-and-bcr-analysis-of-chromium-connect-generated-library-from-10k-human-pbmcs-2-standard)

Download **dataset 2 (CMV + 5K human Tcells )** from the [10x website](https://www.10xgenomics.com/resources/datasets/integrated-gex-totalseqc-and-tcr-analysis-of-connect-generated-library-from-5k-cmv-t-cells-2-standard)

For this tutorial we downloaded the dataset 1 and dataset 2 files from the website and created folders within the 'outs' folder to simulate the directory structure of **cellranger multi** outputs as required by panpipes, as mentioned in our [documentation](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_qc_mm.html#panpipes-sample-submission-file) on sample submission 

```
# Here we show the directory structure of the outs folder for the 5k human CMV + Tcells dataset
# only with the essential files needed by panpipes ingest
tree -L 4 ./outs
outs
|-- multi
|   |-- count
|   |   |-- feature_reference.csv
|   |   |-- raw_feature_bc_matrix
|   |   |   |-- barcodes.tsv.gz
|   |   |   |-- features.tsv.gz
|   |   |   `-- matrix.mtx.gz
|   |   |-- raw_feature_bc_matrix.h5
|   `-- vdj_t
`-- per_sample_outs
    `-- human_cmv
        |-- count
        |   |-- feature_reference.csv
        |   |-- sample_filtered_barcodes.csv
        |   |-- sample_filtered_feature_bc_matrix
        |   |-- sample_filtered_feature_bc_matrix.h5
        |-- metrics_summary.csv
        `-- vdj_t
            |-- filtered_contig_annotations.csv
```

## Submisison and yaml file

We created a sample submission file which will instruct `panpipes` on how to find each modality's path. Download this submission file [here](../ingesting_multimodal_data/submission_file_citeseq_vdj.tsv).

Besides the first column, "sample_id",  the order in which the columns are provided is not fixed, but the column names are fixed! Failing to specify the column names will result in omission of the modality from the analysis and early stopping of the pipeline. 

We find useful to generate the submission file with softwares like Numbers or Excel and save the output as a txt file to ensure that the file is properly formatted.
For more examples please check our [documentation](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_qc_mm.html#panpipes-sample-submission-file) on sample submission files.

As we are ingesting CITE-Seq data, we created a protein metadata table, with information on if the antibody was an isotype or a hashing antibody. it is **important** to note that the  first column equivalent to the first column of cellrangers features.tsv.gz.
Download the protein metadata table for this tutorial [here](../ingesting_multimodal_data/protein_metadata.txt).

## Run panpipes ingest

Now, activate the environment in which you have installed `panpipes`, create a `1_ingest` folder  and configure the `ingest` workflow within that folder

```
mkdir 1_ingest
cd 1_ingest
panpipes ingest config
```

`panpipes ingest config`
This command will generate a config file, `pipeline.yml`. Modify the config file to read in the sample submission file provided. You can find the preconfigured `pipeline.yml` file [here](../ingesting_multiomodal_data/pipeline.yml). 

Please remember to apply the necessary changes in this file to ensure it will run on your computer, and specify:
- the environment in which you're running panpipes, if applicable.
- the path to the custom_genes_file that we use to run `scanpy.score.genes` (we provide an example file in the panpipes' resources folder)

Now, run the full panpipes ingestion workflow with:

` panpipes ingest make full`

The pipeline will write to standard output and to a pipeline.log file about the steps it's running. When it's finished, you will see a  pipeline finished message:

```
2023-11-27 10:07:10,498 INFO main task - Completed Task = 'pipeline_ingest.full' 
2023-11-27 10:07:10,499 INFO main experiment - job finished in 978 seconds at Mon Nov 27 10:07:10 2023 -- 27.26 12.18  0.14  0.52 -- 85d537bd-19b2-41cd-8134-5d615c1342e5
```

## Panpipes ingest outputs

Let's inspect the outputs we have generated with `panpipes ingest` in the '1_ingest' directory"

```
tree -L 2 ./1_ingest
.
|-- 10x_metrics.csv
|-- _downsampled_cell_metadata.tsv
|-- figures
|   |-- atac
|   |-- background
|   |-- barplot_cellcounts_thresholds_filter.png
|   |-- prot
|   |-- rep
|   |-- rna
|   |-- rna_v_prot
|   `-- tenx_metrics
|-- logs
|   |-- assess_background.log
|   |-- concat_bg_mudatas.log
|   |-- concat_filtered_mudatas.log
|   |-- human_cmv_bg_downsampled.log
|   |-- human_pbmc_bg_downsampled.log
|   |-- load_bg_mudatas_human_cmv.log
|   |-- load_bg_mudatas_human_pbmc.log
|   |-- load_mudatas_human_cmv.log
|   |-- load_mudatas_human_pbmc.log
|   |-- plot_qc.log
|   |-- run_dsb_clr.log
|   |-- run_scanpy_qc_prot.log
|   |-- run_scanpy_qc_rep.log
|   |-- run_scanpy_qc_rna.log
|   |-- run_scrublet_human_cmv.log
|   |-- run_scrublet_human_pbmc.log
|   `-- tenx_metrics_multi_aggregate.log
|-- mm_bg.h5mu
|-- mm_cell_metadata.tsv
|-- mm_prot_qc_metrics_per_sample_id.csv
|-- mm_threshold_filter.tsv
|-- mm_threshold_filter_explained.tsv
|-- mm_unfilt.h5mu
|-- pipeline.log
|-- pipeline.yml
|-- scrublet
|   |-- human_cmv_doubletScore_histogram.png
|   |-- human_cmv_scrublet_scores.txt
|   |-- human_pbmc_doubletScore_histogram.png
|   `-- human_pbmc_scrublet_scores.txt
`-- tmp
    |-- human_cmv.h5mu
    |-- human_cmv_raw.h5mu
    |-- human_pbmc.h5mu
    `-- human_pbmc_raw.h5mu
```

The final `MuData` object with computed QC metrics is `mm_unfilt.h5mu`. A `MuData` object without QC metrics for each sample in the ` sample submisison_file` is also available and stored in the `tmp` folder. The metadata of the final `Mudata` object is additionally extracted and saved as a tsv file, `mm_cell_metadata.tsv`. Lastly, the per sample ADT metrics for each antibody are extracted and also saved as a tsv file, `mm_prot_qc_metrics_per_sample_id.csv`.

Moreover a `MuData` object containing information for the background or the raw cellranger output is also created and is `mm_bg.h5mu`. This is either all the barcodes in the raw cellranger or a downsample object dpeending on if the `downsample` param has been set to `True` in the `pipeline.yml`.

The `ingest` pipleine also aggregates and outputs all the cellranger summary metrics for all the samples as a tsv file, `10x_metrics.csv`. 

Additionaly, plots for all modalities for all the summary metrics are individually plotted across all samples and can be found under the `figures/tenx_metrics` directory. This is a nice way to compare and contrast all the samples in a single panpipes run for the relevant metrics.

### 10x metrics plots

With the plots in the `figures/tenx_metrics`, you can evaluate the result of the `cellranger multi` results for your samples. For example, one can check the sequencing quality of the samples by evaluating:
1) the scatter plots of the number of cells vs the median umis in the log10 scale for the RNA modality.
2) the sequencing saturation per vs Number of reads per sample for the RNA modality. 

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/10x_sequencing_saturation_summary.png?raw=true" alt="img1" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/10x_cells_by_UMIs.png?raw=true" alt="img2" width="350"/>
</p>

We also plots different metrics across all samples for all the inputted modalities. It might be useful to examine the number of cells across all the samples for `rna + prot`,`BCR` and `TCR` modalities.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/Gene%20Expression-Cells.png?raw=true" alt="img3" width="250" height="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/VDJ%20B-Estimated%20number%20of%20cells.png?raw=true" alt="img4" width="250" height="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/VDJ%20T-Estimated%20number%20of%20cells.png?raw=true" alt="img5" width="250" height="250"/>
</p>

one can also look at other relevant `10x_metrics` such as the Mean or median reads/UMI counts/genes per cell for the different modalities.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/Gene%20Expression-Median%20UMI%20counts%20per%20cell.png?raw=true" alt="img6" width="250" height="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/Gene%20Expression-Median%20genes%20per%20cell.png?raw=true" alt="img7" width="250" height="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/Antibody%20Capture-Mean%20reads%20per%20cell.png?raw=true" alt="img8" width="250" height="250"/>
</p>


### background plots
The ingest pipeline also plots the barcode rank gene plots for all samples , so that it is easier to contrast and compare them and evaluate the number of barcodes called as cells vs not. These plots can be found in `figures/background`

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/rna_barcode_ranks.png?raw=true" alt="img9" width="350"/>
</p>

Moreover, we also evaluate the downsampled background for the **rna** and **prot** modality seperately and compared to each for all samples. For example a scatter plot visualising the nUMI counts in the empty barcodes (background) vs the cell containing barcodes (foreground) in the protein vs rna.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/scatter_bg_fg_rna_nUMI_prot_nUMI.png?raw=true" alt="img10" width="350"/>
</p>


Additionally, we also plot the highest expressing genes and proteins in all the samples as boxplots and heatmaps. This helps you evaluate potential sources of ambient contamination in the samples, like heamoglobin contamination or high mitochondrial gene expression or high backgroudn expression of particular antibodies.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/highest_expr_genes_rna_background.png?raw=true" alt="img11" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/heatmap_background_sample_id_prot_top_expressed.png?raw=true" alt="img12" width="350"/>
</p>


### RNA QC plots
`panpipes ingest` workflow computes qc metrics for each modality given as input. For a multimodal sample it may be useful to check `RNA` metrics under the `figures/rna` such as number of genes detected in cells vs the total UMI counts , percentage of mitochondrial reads per cell vs library size.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/scatter_sample_id_rna-nUMI_v_rna-genes.png?raw=true" alt="img13" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/scatter_sample_id_rna-log1p_nUMI_v_rna-log1p_genes.png?raw=true" alt="img14" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/scatter_sample_id_rna-nUMI_v_rna-pct_mt.png?raw=true" alt="img15" width="350"/>
</p>


### Protein QC and normalisation plots
For the `PROT` metrics it might be useful to check the total counts and mean counts of per ADT across samples. The figures can be found under 
 `figures/prot`.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/boxplot_sample_id_total_counts.png?raw=true" alt="img16" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/boxplot_sample_id_mean_counts.png?raw=true" alt="img17" width="350"/>
</p>

As for the `prot` modality `clr` and `dsb` normalisation can also be performed in the `panpipes ingest` step, we can also visualise the normalised ridge plots for 1) clr & 2) dsb normalisation for each of the ADTs for each sample in the `sample_submission` file.

The `dsb` normalisation corrects for background `ADT` noise and cell-cell technical cell-to-cell variations and can help to identify a cleaner expression protein signal in comparison to the `clr` normalisation. Here we show the clr and dsb ridge plots for the `human_cmv` dataset.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/human_cmv_clr_ridgeplot.png" alt="img18" width="350"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/human_cmv_dsb_ridgeplot.png?raw=true" alt="img19" width="350"/>
</p>


### Repertoire QC plots
For the QC of of `VDJ` data, we utilise the **scirpy** python package function `scirpy.tl.chain_qc` to perform QC based on the receptor-chain pairing configuration and categorise the cells into their receptor types and subtypes. This classification of the cells can be checked in the plots under `figures/rep`.

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/barplot_group_abundance_receptor_type.png?raw=true" alt="img20" width="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/barplot_group_abundance_bcr_receptor_subtype.png?raw=true" alt="img21" width="250"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/barplot_group_abundance_tcr_receptor_subtype.png?raw=true" alt="img22" width="250"/>   
</p>

### Multiple Filter threshold plots

To aid with the filtering of the data, we also produce outputs that simulate common filtering scenarios, the height of the bar shows the percentage of cells retained if the threshold is applied. 

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/da_ingest_multimodal/docs/ingesting_multimodal_data/barplot_cellcounts_thresholds_filter.png?raw=true" alt="drawing" width = "500" height = "450"/>
</p>


Users can also inspect the unfiltered mudata generated file by reading it in a python session:

```
import muon as mu
mu.read("mm_unfilt.h5mu")
```

We have demonstrated how to run the `ingest` workflow on multimodal (CITE-Seq + VDJ) data with multiple samples. Filtering of cells and genes is not applied in the `ingest` workflow but in the `preprocess`. Inspecting these output should help the user to choose appropriate filters for their data!
