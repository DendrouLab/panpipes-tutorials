Clustering tutorial
===================

After the `integration` pipeline, we can run clustering in order to discover the cell-type composition of our dataset in an unsupervised way.

Let's create a clustering folder where we can run our workflow.

*Note: if you are in one of the integration folders, you can go up one directory and set yourself at the same level; we prefer to create the clustering folder inside the integration folder after choosing the integrations  with* `panpipes make merge_integration` 
*As usual, customize your project folder structure as works best for you!*

```
mkdir clustering && cd $_
```

now run `panpipes clustering config`. Inspect and customize the yml (or use the yml we provide in [clustering](../clustering/pipeline.yml))
In the config file, we specify we want to use the teaseq object we have generated in the integration folder.

It's a rather simple setup, we simply specify 


Now run `panpipes clustering make full --local`  
You will find in the outputs:
 - the clustered h5mu object with all the tested combination of clustering parameters on the `.obs` of the respective modality (and the clustering of the multimodal representation in the outer `.obs`)
 - a metadata file with all the modality-specific `.obs` concatenated
 - a folder for each modality
  
    inside of these:

    - the computed umap coordinates with the specified choice of parameters
    - one directory for each combination of clustering parameters 
  
  Finally, each of the clustering directories contains plots and txt files for the marker analysis. We run the marker analysis using the clustering results and ANY of the modalities. That means you can discover protein markers for clusters that were calculated on RNA, or on ATAC, and viceversa.


