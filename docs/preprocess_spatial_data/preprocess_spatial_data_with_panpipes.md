# Preprocessing spatial data with Panpipes

The `preprocess_spatial` workflow expects one or multiple `SpatialData` objects as input. The workflow filters the data, followed by normalization, HVG selection, and PCA computation. The steps of the workflow are explained in greater detail [here](https://panpipes-pipelines.readthedocs.io/en/latest/workflows/preprocess_spatial.html).

For all the tutorials, we will append the `--local` command which ensures that the pipeline runs on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a cluster.


## Directories and data

For the preprocessing tutorial, we will work in the main `spatial` directory and create a `preprocess` directory for the preprocessing: 

```
# mkdir spatial # <- if you don't have the spatial directory already 
# cd spatial
mkdir preprocess
cd preprocess
```

In this tutorial, we will use the output `Spatialdata` objects of the [Visium ingestion tutorial](../ingesting_visium_data/Ingesting_visium_data_with_panpipes.md). Namely, the `SpatialData` files saved in `spatial/ingestion/qc.data/`:


```
spatial 
├── preprocess
└── ingestion
    ├── data
    ├── figures
    ├── logs
    ├── qc.data # MuDatas with QC metrics 
    │	├── V1_Human_Heart_unfilt.zarr
    │	└── V1_Human_Lymph_Node_unfilt.zarr
    ├── tmp 
    ├── pipeline.log
    ├── pipeline.yml
    ├── sample_file_qc_spatial.txt
    ├── V1_Human_Heart_cell_metadata.tsv 
    └── V1_Human_Lymph_Node_cell_metadata.tsv
```

The `preprocess_spatial` workflow allows you to preprocess one or multiple `SpatialData` objects **of the same assay, i.e. Visium, Vizgen, or Xenium,** in one run. For that, the workflow reads in all `.zarr` files of the input directory. The `SpatialData` objects of the input directory are then **preprocessed with the same specified parameters**.
 

## Edit yaml file 

In `spatial/preprocess`, create the pipeline.yml and pipeline.log files by running `panpipes preprocess_spatial config` (you potentially need to activate the conda environment with `conda activate pipeline_env` first!). 
Modify the yaml file, or simply use the [pipeline.yml](pipeline_yml.md) that we provide (you potentially need to add the path of the conda environment in the yaml). Note, that the filtering step is **optional**. You can avoid filtering by setting the `run` parameter under `filtering` to `False`. The pipeline will then only normalize the data, compute HVGs and run PCA.  



## Run Panpipes

Run the full workflow with `panpipes preprocess_spatial make full --local`

Once `Panpipes` has finished, the `spatial/preprocess` directory will have the following structure:

```
preprocess
├── figures
│   └── spatial
│       ├── pca_variance_ratio.V1_Human_Heart.png
│       ├── pca_variance_ratio.V1_Human_Lymph_Node.png
│       ├── pca_vars.V1_Human_Heart.png
│       ├── pca_vars.V1_Human_Lymph_Node.png
│       ├── spatial_spatial_total_counts.V1_Human_Heart.png
│       ├── spatial_spatial_total_counts.V1_Human_Lymph_Node.png
│       ├── violin_obs_total_counts_.V1_Human_Heart.png
│       ├── violin_obs_total_counts_.V1_Human_Lymph_Node.png
│       ├── violin_var_total_counts.V1_Human_Heart.png
│       └── violin_var_total_counts.V1_Human_Lymph_Node.png
├── filtered.data
│   ├──V1_Human_Heart_filtered.zarr  
│   └── V1_Human_Lymph_Node_filtered.zarr
├── logs
│   ├── filtering.V1_Human_Heart_.log  
│   ├── filtering.V1_Human_Lymph_Node_.log  
│   ├── postfilterplot.V1_Human_Heart.log       
│   ├── postfilterplot.V1_Human_Lymph_Node.log 
│   ├── st_preprocess.V1_Human_Heart.log
│   └── st_preprocess.V1_Human_Lymph_Node.log
├── pipeline.log
├── pipeline.yml
└── tables
│   ├── V1_Human_Heart_filtered_cell_counts.csv
│   ├── V1_Human_Heart_filtered_filtered_cell_metadata.tsv
│   ├── V1_Human_Lymph_Node_filtered_cell_counts.csv
│   └── V1_Human_Lymph_Node_filtered_filtered_cell_metadata.tsv
```

You can find the final `SpatialData` objects in the `spatial/preprocess/filtered.data` folder. Additionally, the metadata of the filtered `Spatialdata` objects is saved as tsv files in the `spatial/preprocess/tables` directory, together with csv-files containing the number of spots/cells after filtering.

Post-filter plots are stored in `spatial/preprocess/figures/spatial`.  The plots include visualizations of the spatial embeddings, as well as violin plots: 

<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/preprocess_spatial_data/spatial_spatial_total_counts.V1_Human_Lymph_Node.png?raw=true" alt="Spatial embedding, total_counts" width="300"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/preprocess_spatial_data/violin_obs_total_counts_.V1_Human_Lymph_Node.png?raw=true" alt="Violin plot, total_counts" width="300"/>
</p>

The PCA and the elbow plot are also plotted: 
<p align="center">
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/preprocess_spatial_data/pca_variance_ratio.V1_Human_Lymph_Node.png?raw=true" alt="PCA variance ratio" width="270"/>
<img src="https://github.com/DendrouLab/panpipes-tutorials/blob/main/docs/preprocess_spatial_data/pca_vars.V1_Human_Lymph_Node.png?raw=true" alt="PCA" width="270"/>
</p>



*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*