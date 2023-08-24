## Ingesting merfish spatial data with panpipes




Create a main `spatial` directory to start processing samples and inside it, `ingestion_merfish`.

```
mkdir spatial & cd $_
mkdir ingestion_merfish & cd $_
```
You can download the input data we will use for this example [here](https://info.vizgen.com/mouse-brain-map?submissionGuid=a66ccb7f-87cf-4c55-83b9-5a2b6c0c12b9) 

configure the pipeline using

`panpipes qc_spatial config` and customize the `pipeline.yml`. we provide this yaml in this [repository](../ingesting_processing_merfish_data/)













