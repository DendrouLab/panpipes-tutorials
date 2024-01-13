<style>
  .parameter {
    border-top: 4px solid lightblue;
    background-color: rgba(173, 216, 230, 0.2);
    padding: 4px;
    display: inline-block;
    font-weight: bold;
  }
</style>

# pipeline.yml for ingestion workflow

## Compute resource options

* <p class="parameter">resources</p>
	
  Computing resources to use, specifically the number of threads used for parallel jobs. Specified by the following three parameters:

  * <p class="parameter">threads_high</p>
	
  	For each thread, there must be enough memory to load all your input files at once and create the MuData object. Defaults to 1.
  * <p class="parameter">threads_medium</p>
	
  	For each thread, there must be enough memory to load your mudata and do computationally light tasks. Defaults to 1.
  * <p class="parameter">threads_low</p>
	
  	For each thread, there must be enough memory to load text files and do plotting, requires much less memory than the other two. Defaults to 1.

#TODO: Check if it really is for each thread

* <p class="parameter">condaenv</p>
	 Path to conda environment to use for running panpipes, leave blank if running native or your cluster automatically inherits the login node environment

## Loading and concatenating data options
### Project name and data format
* <p class="parameter">project</p>
	
  Project name. Defaults to "test".
* <p class="parameter">sample_prefix</p>
	
  Prefix for sample names. Defaults to "test".
  	#TODO: Is this the right definition?
* <p class="parameter">use_existing_h5mu</p>
	
  If you have an existing MuData object (.h5mu file) that you want to run panpipes on, then store it in the folder where you intend to run the workflow, and call it ${sample_prefix}_unfilt.h5mu where ${sample_prefix} = sample_prefix argument above
	#TODO: specify which folder. The one of the workflow? What is the $ in the name?
* <p class="parameter">submission_file</p>
	
  Submission file format. For ingest, the submission file must contain the follwing columns: sample_id  rna_path  rna_filetype  (prot_path  prot_filetype tcr_path  tcr_filetype etc.)
  	An example submission file can be found at resources/sample_file_mm.txt
	#TODO: What is meant by format?
* <p class="parameter">metadatacols</p>
	
  Which metadata cols from the submission file do you want to include in the anndata object as a comma-separated string e.g. batch,disease,sex
* <p class="parameter">concat_join_type</p>
	
  Specifies which concat join is performed on the MuData objects. We recommended inner. See the [AnnData documentation](https://anndata.readthedocs.io/en/latest/concatenation.html#inner-and-outer-joins) for details. Defaults to inner.

### Modalities in the project
the qc scripts are independent and modalities are processed following this order. Set to True to abilitate modality(ies). 
Leave empty (None) or False to signal this modality is not in the experiment. By default, all modalities are set to False.

* <p class="parameter">modalities:</p>
    * <p class="parameter">rna</p>
    * <p class="parameter">prot</p>
    * <p class="parameter">bcr</p>
    * <p class="parameter">tcr</p>
    * <p class="parameter">atac</p>
    
### Integrating barcode level data
Integrating barcode level data, e.g. demultiplexing with hashtags, chemical tags or lipid tagging

if you have cell level metadata such as results from a demultiplexing algorithm, you can incorporate it into the mudata object, this should be stored in one csv containing 2 columns, barcode_id, and sample_id, which should match the sample_id column in the submission file.

* <p class="parameter">barcode_mtd</p>
    * <p class="parameter">include</p> Defaults to True.
    * <p class="parameter">path</p> #TODO: What is meant by path?
    * <p class="parameter">metadatacols</p> #TODO: What is meant by metadatacols?

### Loading prot data - additional options
* <p class="parameter">protein_metadata_table</p>
    
  As default the ingest will choose the first column of the cellranger features.tsv.gz. To merge extra information about the antibodies e.g. whether they are hashing antibodies or isotypes. Create a table with the first column equivalent to the first column of cellrangers features.tsv.gz and specify it below (save as txt file)
  Make sure there are unique entries in this column for each prot. Include a column called isotype (containing True or False) if you want to qc isotypes. Include a column called hashing_ab (containing True or False) if your dataset contains hashing antibodies which you wish to split into a separate modality

* <p class="parameter">index_col_choice</p>

    If you want to update the mudata index with a column from protein_metadata_table, specify here
    If there are overlaps with the rna gene symbols, then you might have trouble down the line
    It is recommended to add something to make the labels unique e.g. "prot_CD4"

* <p class="parameter">load_prot_from_raw</p>
If providing separate prot and rna 10X outputs, then the pipeline will load the filtered rna 10X outputs and the raw prot 10X counts
we assume in most cases that we want to treat filtered rna barcodes as the "real" cells
and we want out prot assays barcodes to match rna.
Defaults to False.

* <p class="parameter">subset_prot_barcodes_to_rna</p>
In which case set subset_prot_barcodes_to_rna: True (which is also the default setting) if explicitly set to False, the full prot matrix will be loaded into the mudata obect. Defaults to False.

## Quality Control (QC) options
### 10X cellranger files processing


<p class="parameter">XYZ</p>