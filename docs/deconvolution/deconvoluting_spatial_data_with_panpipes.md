Deconvoluting spatial data with panpipes
----------------------------------------

The `deconvolution_spatial` workflow provides the possibility to run deconvolution for spatial data. 
Let's run the following commands to create the directory `deconvolution` and to create the `pipeline.yml` and `pipeline.log` files: 
```
mkdir deconvolution & cd $_
panpipes deconvolution_spatial config --local
```

As with the other workflows of panpipes, the deconvolution uses muData objects and expects those as input. For each spatial slide, it expects one muData, with the spatial assay saved under `mudata.mod['spatial']`. For the reference scRNA-Seq data, it expects one muData, with the RNA assay saved under `mudata.mod['rna']`. 
With panpipes, we can run the deconvolution for multiple slides and the same reference in parallel. For that, the muDatas of the spatial data need to be located in the same folder. The pipeline will treat every muData in that folder as a spatial mudata. Therefore, the reference muData should be saved at a different location. Let's look at an example folder structure: 

```
deconvolution
|──	data
	|── sc_reference.h5mu
	|── spatial_data
	|	|── slide1.h5mu
	|	|── slide2.h5mu
```

In the pipeline.yml file, we can now specify the paths of the input data, for our example `input_spatial = ./data/spatial_data` and `input_singlecell = ./data/sc_reference.h5mu`.


## Cell2Location

With the `deconvolution_spatial` workflow we can currently only run [Cell2Location](https://doi.org/10.1038/s41587-021-01139-4). 

Before fitting the reference and the spatial mapping models, the workflow performs gene selection. 
For the gene selection, we have two possibilities. One possibility is to use a pre-defined gene set. But please note, that all genes of that gene list need to be present in both, spatial slides and scRNA-Seq reference. The other possibility is to run [gene selection according to Cell2Location](https://cell2location.readthedocs.io/en/latest/cell2location.utils.filtering.html).

The gene selection and the parameters of the models should be specified in the pipeline.yml file. 
After specifying all parameters, we can run the pipeline with `panpipes deconvolution_spatial make full --local`.

	
In the folder `./cell2location.output` a folder for each slide will be created containing the following outputs: 
* muData containing the spatial data together with the posterior of the spatial mapping model
* muData containing the reference data together with the posterior of the reference model
* A csv-file `Cell2Loc_inf_anver.csv` containing the estimated expression of every gene in every cell type 
* If `save_models = True`, the reference model and the spatial mapping model 
	
In the `./figures/Cell2Location` folder, there will be a folder for each slide created. Each folder will contain the following plots: 
* If gene selection according to Cell2Location is performed: a plot of the gene filtering
* For both models:
  * QC plots
  * ELBO plot
* Spatial plot where the spots of the slide are colored by the estimated cell type abundances 



*Note: We find that keeping the suggested directory structure (one main directory by project with all the individual steps in separate folders) is useful for project management. You can of course customize your directories as you prefer, and change the paths accordingly in the `pipeline.yml` config files!*
