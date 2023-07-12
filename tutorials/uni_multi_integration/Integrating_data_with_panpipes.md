## Integrating data with panpipes

You have ingested, qc'd and filtered your single cell data and you have now created a `.h5mu` object with the outputs.
It's time to run integration before applying clustering to discover cell-types in your data.

Go to your previously created `teaseq` directory and create a new folder to run `panpipes integration`

```
# if you are in teaseq/qc_mm
# cd ..
mkdir integration & cd $_
```

