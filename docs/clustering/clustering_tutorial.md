Clustering tutorial
===================

After the `integration` pipeline, we can run clustering in order to discover in an unsupervised way the cell-type composition in our dataset.

Let's create a clustering folder where we can run our workflow.

*Note: if you are in one of the integration folders, you can go up one directory and set yourself at the same level; we prefer to create the clustering folder inside the integration folder after choosing the integrations  with* `panpipes make merge_integration` 
*As usual, customize your project folder structure as works best for you!*

```
mkidr clustering & cd $_
```

now run `panpipes clustering config` . Inspect and customize the yml (or use the yml we provide in [clustering](../clustering/pipeline.yml))

and run `panpipes clustering make full --local`  
You will find in the outputs:
 - a folder for each modality
   - inside, each resolution 

