The `preprocess_spatial` workflow expects as input one or multiple `mudata` each with a `spatial` slot where the spatial assay is saved. See [ingestion tutorial](../ingesting_spatial_data/Ingesting_spatialdata_with_panpipes.md) for an example.

Following from the ingestion tutorial, we're in the main `spatial` directory. We can create a `preprocess` directory where we will run the filtering, followed by scaling, normalization, and dimensionality reduction.

```
# if you are in spatial/ingestion
# cd ..
mkdir preprocessing & cd $_

```

In this folder, run `panpipes preprocess_spatial config` to create the pipeline.yml configuration file.
Modify the file, or just use the one we provide in [filtering spatial data](../filtering_spatial_data/pipeline.yml)


Now run `panpipes preprocess_spatial make full --local`


*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*
