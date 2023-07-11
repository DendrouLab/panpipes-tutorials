## Filtering data with panpipes

You have ingested your single cell data and you have created a `.h5mu` object storing calculated qc metrics.
It's time to filter the cells to make sure to exclude bad quality cells for downstream analysis.

go to your previously created `teaseq` directory and create a new folder to run `panpipes preprocess`

```
# if you are in teaseq/qc_mm
# cd ..
mkdir preprocessing & cd $_
```

In here, run `panpipes preprocess config --local`, which will generate again a `pipeline.yml` file for you to customize. We provide this yml in the [filtering_data](https://github.com/DendrouLab/panpipes_reproducibility/tree/main/tutorials/filtering_data) folder in this repo.

Open the yml file to inspect the parameters choice. 




