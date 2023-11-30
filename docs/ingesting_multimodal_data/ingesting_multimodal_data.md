Ingesting multimodal (CITE-Seq + VDJ) datasets from cellranger outputs 
===================

Panpipes can read files from different cellranger outputs. Here we will showcase the example of  two multmodal (CITE-Seq + VDJ) datasets from the 10x website while ingesting cellranger multi outputs. 

Download **dataset 1 ( 10k human PBMCs)** from the [10x website](https://www.10xgenomics.com/resources/datasets/integrated-gex-totalseq-c-and-bcr-analysis-of-chromium-connect-generated-library-from-10k-human-pbmcs-2-standard)

Download **dataset 2 ( 5K human Tcells + CMV)** from the [10x website](https://www.10xgenomics.com/resources/datasets/integrated-gex-totalseqc-and-tcr-analysis-of-connect-generated-library-from-5k-cmv-t-cells-2-standard)

For this tutorial we downloaded the dataset 1 and dataset 2 files from the website and created folders within the 'outs' folder to simulate the directory structure of **cellranger multi** outputs as required by panpipes, as mentioned in our [documentation](https://panpipes-pipelines.readthedocs.io/en/latest/usage/setup_for_qc_mm.html#panpipes-sample-submission-file) on sample submission 

```
# Here we show the directory structure of the outs folder for dataset 1 (human PBMC)
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
We created a sample submission file which will instruct `panpipes` on how to find each modality's path. Download this submission file [here](../ingesting_multimodal_data/submission_file_citeseq_vdj.tsv).
