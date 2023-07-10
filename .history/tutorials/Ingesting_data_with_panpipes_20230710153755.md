## Ingesting data with panpipes

Panpipes is a single cell multimodal analysis pipeline with a lot of functionalities to streamline and speed up your single cell projects.

Arguably, the most important part of a pipeline is the ingestion of the data into a format that allows efficient storage and agile processing. We believe that `AnnData` and `MuData` offer all those advantages and that's why we built `panpipes` with these data structure at its core. 
Please check the [`scverse` webpage](https://scverse.org/) for more information!

We will give you a couple of examples of reading data from a 10X directory or directly from existing anndata objects, but we offer functionalities to read in any tabular format and assay-specific data types (check out Spatial Transcriptomics and Repertoire analysis)


### Start from 10X directories

lorem ipsum

### Start from pre-existing h5ad objects

Please download the input data that we have provided [here](). It's a random subset of cells from the [teaseq datasets]() that we also used for the `panpipes` paper.

You should find three `.h5ad` objects in this directory, one for each modality of the teaseq experiment, namely `rna`, `adt` and `atac`.

In order to ingest the data, we have to tell panpipes the paths to each anndata.
Create a csv file like the one we provide in the [docs](), if you have cloned this repo, you should have it under `sample_file_qc.txt`.

