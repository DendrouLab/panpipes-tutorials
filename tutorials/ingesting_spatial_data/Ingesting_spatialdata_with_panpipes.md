## Ingesting spatial data with panpipes
You can now generate consistent analyses of your spatial transcriptomics data!

Let's run through an example of reading data from  `10X visium` outputs (`vizgen-merfish` tutorials coming soon!) 
For all the tutorials we will append the `--local` command which ensures that the pipeline runs on the computing node you're currently on, namely your local machine or an interactive session on a computing node on a cluster.

### Start from 10X directories

Create a main `spatial` directory to start processing samples.
You can download the input datasets we will use for this example from the 10x database.
[Human Lymph node](https://support.10xgenomics.com/spatial-gene-expression/datasets/1.0.0/V1_Human_Lymph_Node)
[Human Heart](https://www.10xgenomics.com/resources/datasets/human-heart-1-standard-1-0-0)

(These are the same datasets you'd be able to dowload using `sc.datasets.visium_sge(sample_id="V1_Human_Lymph_Node")`)

Inside the `spatial` directory, you should have a directory with all the data you downloaded 

```
data
├── V1_Human_Heart
│   ├── V1_Human_Heart_spatial.tar.gz
│   ├── filtered_feature_bc_matrix.h5
│   └── spatial
│       ├── aligned_fiducials.jpg
│       ├── detected_tissue_image.jpg
│       ├── scalefactors_json.json
│       ├── tissue_hires_image.png
│       ├── tissue_lowres_image.png
│       └── tissue_positions_list.csv
└── V1_Human_Lymph_Node
    ├── V1_Human_Lymph_Node_spatial.tar.gz
    ├── filtered_feature_bc_matrix.h5
    └── spatial
        ├── aligned_fiducials.jpg
        ├── detected_tissue_image.jpg
        ├── scalefactors_json.json
        ├── tissue_hires_image.png
        ├── tissue_lowres_image.png
        └── tissue_positions_list.csv
```

Now create a csv file like the one we provide in the [tutorials](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/ingesting_spatial_data), if you have cloned this repo, you should have it under `submission_10x_file.txt`.

Now in `spatial` call `panpipes qc_spatial config`.
this will generate a `pipeline.log` and a `pipeline.yml` file.

Modify the `pipeline.yml` with custom parameters or simply replace with the one we provide in [tutorials](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/ingesting_spatial_data)

Run `panpipes qc_spatial make full --local` to ingest your visium datasets.

This command will produce one mudata for each input sample in the submission file and save them in the "tmp" directory. In this workflow we have decided to process individual ST sections instead of concatenating them at the beginning, like you saw for cell-suspension datasets. We offer the option to concatenate the filtered objects in the `xxx` spatial workflow just before the deconvolution step, in the future we will implement the advanced functionalities of [SpatialData](https://spatialdata.scverse.org/en/latest/tutorials/notebooks/notebooks.html) to deal with multi-sample ST datasets.



#### [Next: filtering visium data using `panpipes preprocess_spatial`](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/filtering_spatial_data/filtering_spatial_data_with_panpipes.md)













