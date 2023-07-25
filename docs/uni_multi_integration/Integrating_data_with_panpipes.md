## Integrating data with panpipes

You have ingested, qc'd and filtered your single cell data and you have now created a `.h5mu` object with the outputs.
It's time to run integration before applying clustering to discover cell-types in your data.

Go to your previously created `teaseq` directory and create a new folder to run `panpipes integration`

```
# if you are in teaseq/preprocessing
# cd ..
mkdir integration & cd $_
```
You can now create the pipeline.yml by launching `panpipes integration config`. Again, we provide the preconfigured yml in the [integration]() folder.
As we did before, link the preprocessed h5mu object in the present directory where the 

```
ln -s ../preprocessing/teaseq.h5mu .
```

For this example, we will run some uni and multimodal integration methods. 
In the yml we specify:



```
for rna: harmony,bbknn,scvi
for prot: harmony, bbknn
for atac: harmony, bbknn

and for multimodal integration:
totalVI: rna,prot
WNN: rna,atac,prot 
MOFA+: rna,atac,prot

```
Also note that by default, `panpipes` will always produce the uncorrected unimodal objects by running neighbours and umap on the baseline dimensionality reduction, PCA or LSI depending on the modality. You can also leave blank the batch correction arguments, and the uncorrected objects will be the only outputs.

For background on each method please consider reading the best practices for cross-modal single cell paper[REF], the benchmarks on batch correction and multimodal integration[REF]

Run the integration workflow with `panpipes integration make full --local`
Once the pipeline finishes, you will find the uni or multimodal integrated objects in a `tmp` folder, along with relevant auxillary files such as the scvi trained model in `batch_correction/scvi_model` and plots are saved in `figures`.
In the paper, we showcase how WNN offers the flexibility to integrate modalities that have individually been batch-corrected.
To showcase this scenario, we will use the last functionality of `panpipes integration` workflow, the `make merge_integration`. This task should be run after you have inspected the results of the integration and have decided which of the applied corrections you want to keep, for both uni and multimodal approaches.

let's modify the `pipeline.yml` file and keep :

```
# ----------------
# Make final object
# ----------------
## Final choices: Leave blank until you have reviewed the results from running
# panpipes integration make full
# choose the parameters to integrate into the final anndata object
# then run
# panpipes integration make merge_integration
final_obj:
  rna:
    include: True
    bc_choice: scvi
  prot:
    include: True
    bc_choice: harmony
  atac:
    include: False
    bc_choice: harmony
    n_neighbors:
    umap_min_dist:
  multimodal:
    include: True
    bc_choice: WNN

```
and run `panpipes integration make merge_integration --local`.

once this is finished, you will find a new object in the main directory, namely `teaseq_corrected.h5mu`.
let's create a new directory 

```
mdkir sub_integration & cd $_
ln -s ../teaseq_corrected.h5mu teaseq_temp.h5mu
cp ../pipeline.yml .
```

now let's modify the name of the input file to `teaseq_temp.h5mu`. 
Let's also set to false all the modalities batch_correction algorithms and change the `wnn` parameters in the yaml file to instruct wnn to run on the batch corrected data from the `prot` and `atac` modalities in the h5mu input object.
To compare the new vs the nobatch wnn run we have done in `integration`, we can create a `batch_correction` directory in this subfolder and link the original file to this location by taking care of adding a string that will distinguish the original wnn run from this new one.
For example: 


```
mkdir batch_correction & cd $_
ln -s ../../batch_correction/umap_multimodal_wnn.csv umap_multimodal_wnnnobatch.csv
```

also, to avoid re-running the unimodal no_correction runs, we can link the individual modalities `batch_correction/umap_*_none.csv` in the same way.

let's now run again `panpipes integration make full --local`

The pipeline will pick the new requirement for wnn and create a new wnn run with the desired batch corrections for each modality, and since we have linked the previous `wnn` correction in this subdirectory, it will generate the outputs (check the figures folder for plots and scores) to compare `wnn`` with and without batch correction.
