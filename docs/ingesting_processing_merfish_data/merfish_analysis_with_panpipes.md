## Ingesting merfish spatial data with panpipes

Create a main `spatial` directory to start processing samples and inside it, `ingestion_merfish`.

```
mkdir spatial & cd $_
mkdir ingestion_merfish & cd $_
```
You can download the input data we will use for this example [here](https://info.vizgen.com/mouse-brain-map?submissionGuid=a66ccb7f-87cf-4c55-83b9-5a2b6c0c12b9) 

configure the pipeline using

`panpipes qc_spatial config` and customize the `pipeline.yml`. We provide this yaml in this [repository](../ingesting_processing_merfish_data/)

To create the mudata object with calculation of qc metadata, run the full pipeline with `panpipes qc_spatial make full --local` and inspect the plots in `figures/spatial` 

### Preprocessing merfish data

After you have ingested the data into a `mudata` you can apply further processing such as filtering by obs or vars and normalization, scaling and dimensionality reduction.  
Go back to the main directory `spatial` and create a new directory for the processing

```
cd .. # go back to the main dir 
mkdir process_merfish & cd $_
```

Configure the pipeline using

`panpipes preprocess_spatial config` and customize the `pipeline.yml`. We provide this yaml in this [repository](../ingesting_processing_merfish_data/processing_pipeline.yml) (remember to save this yml as `pipeline.yml`` in this directory so that `panpipes`` can read it!)

We use very minimal processing, filtering by minimum number of genes and requesting the normalization by analytic Pearson residuals (introduced in the sctransform paper).

Run the full pipeline with `panpipes preprocess_spatial make full --local`

Once `panpipes` has finished you will have a directory with the following structure:

```
preprocess_merfish
├── figures
│   └── spatial
│       ├── histograms.Sample3.png
│       ├── pca_variance_ratio.Sample3.png
│       ├── pca_vars.Sample3.png
│       ├── spatial_spatial_n_genes_by_counts.Sample3.png
│       ├── spatial_spatial_total_counts.Sample3.png
│       ├── violin_obs_n_genes_by_counts_.Sample3.png
│       ├── violin_obs_total_counts_.Sample3.png
│       └── violin_var_total_counts.Sample3.png
├── filtered.data
│   └── Sample3_filtered.h5mu
├── logs
│   ├── filtering.Sample3_.log
│   ├── postfilterplot.Sample3.log
│   └── st_preprocess.Sample3.log
├── pipeline.log
├── pipeline.yml
└── tables
    ├── Sample3_filtered_cell_counts.csv
    └── Sample3_filtered_filtered_cell_metadata.tsv
```

The filtered, normalized and scaled object is in the `filtered.data` folder, with some useful visualization to inspect in the `figures`, such as the distribution of counts and genes per in the `histograms.Sample3.png` or overlayed on the tissue slide `spatial_n_genes_by_counts.Sample3.png`.

### Clustering merfish data

Let's try to capture clusters based on spot-transcriptional similarity using the standard `scanpy` single best practices, using the `panpipes clustering` workflow.

```
cd .. # go back to the main dir 
mkdir process_merfish & cd $_
```

To configure the clustering pipeline run `panpipes clustering config`. 
Check out the precompiled yml we provide in the [docs section](../ingesting_processing_merfish_data/clustering_pipeline.yml)

Every modality is set to False except for the spatial, and we need to declare the input mudata path.

In the `yml` we also specify that we aim to compute the knn neighborhood graph on the precomputed PCA (from the preprocessing step), and that we will run leiden clustering with 3 resolutions and find the marker genes on the normalized data that is stored in the layer `norm_pearson_resid` . 

Now run `panpipes clustering make full --local` .


*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*








