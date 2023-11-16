Reference mapping
===================

Reference mapping is the process of aligning a new single cell dataset to an existing single cell atlas generated with a deep learning model that allows to see the new data in the context of a pre-genererated latent space (using the reference atlas).

Panpipes supports the use of `scvi-tools` for constructing and leveraging reference models to do query to reference mapping (scvi, scanvi and TotalVI).

We will show how panpipes allows to map the same query to two different references models, an scvi and a scanvi reference model, respectively.
In this case, the underlying data used for the two reference models is exactly the same, but we're going to use it just as an example of panpipes' functionality to map the same query in parallel to two reference models. 

Download the data for the reference and query used in this example:

Now create a directory where you will run the refmap workflow:

```
mkdir reference_mapping && cd reference_mapping
mkdir data.dir
mkdir models
```
ensure the data and the models you downloaded are in the data.dir and models folders respectively


```
ls data.dir
pancreas_querydata.h5ad
pancreas_refdata.h5mu

ls models
pancreas_model/model.pt
pancreas_model_scanvi/model.pt
```

As with all the other worflows you can configure the refmap workflow using:

`panpipes refmap config`

which will generate the `pipeline.yml` file that is used as input. 
We provide the pipeline yml file for this tutorial [here](../../../docs/refmap_pancreas/).


In the config file, we only have to specify the path to the query and whether the query has a batch covariate and if it has a celltype annotation column:

```
#----------------------
# query dataset
#----------------------
query: pancreas_query.h5ad
# we support raw10x formats, or preprocessed quality filtered mudata or anndata as input query
modality: rna
# if you supplied a mu data, specify the modality to be used
# only RNA modality supported by current models!
# does your query data have a batch effect, what column?
query_batch:
# does your query have a celltype annotation you want to use to compare to
# the transferred label?
# leave empty if not
query_celltype: celltype

```


For the reference models, we specify the path to each model, and since we have the reference data we can also supply it. 

```
#----------------------
# scvi tools params
#----------------------
# specify one or more reference models that you would like to use as reference
# you can use your own reference that you built using pipeline_integration here
# leave blank for no model specification

# specify the reference anndata/mudata with RNA. you need this only if you want to calc umap on both reference and query data.
# otherwise leave blank and only provide model paths
reference_data: path_to_mudata 
[...]
# Specify the full path to the model, i.e. Path/to/scanvi/model.pt
# if not running the method, remove the dummy - path_to_xxx and leave blank
scvi:
  - path_to_scvi
scanvi: 
  - path_to_scanvi 

```

Panpipes doesn't require the reference anndata.mudata to run, the reference models are suffucuent. This is to mimic the scenario in which the reference is too big to be shared or there are some privacy concernts. 
However, if the reference data is supplied, `refmap` will use it to update the latent representation and plot the query and reference cells in the same umap.


