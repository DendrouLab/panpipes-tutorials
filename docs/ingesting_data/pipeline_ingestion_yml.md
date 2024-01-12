<style>
  .parameter {
    border-top: 4px solid lightblue;
    background-color: rgba(173, 216, 230, 0.2);
    padding: 4px;
    display: inline-block;
    font-weight: bold;
    color: cyan;
  }
</style>

# pipeline.yml for ingestion workflow

## Compute resource options

* Computing resources:
	<p class="parameter">resources</p>
	
  Number of threads used for parallel jobs. Specified by the following three parameters:

  * <p class="parameter">threads_high</p>
	
  	???For each thread???, there must be enough memory to load all your input files at once and create the MuData object. Defaults to 1.
  * <p class="parameter">threads_medium</p>
	
  	??For each thread???, there must be enough memory to load your mudata and do computationally light tasks. Defaults to 1.
  * <p class="parameter">threads_low</p>
	
  	??For each thread???, there must be enough memory to load text files and do plotting, requires much less memory than the other two. Defaults to 1.

* Running environment:
	<p class="parameter">condaenv</p>
	 path to conda env, leave blank if running native or your cluster automatically inherits the login node environment

## Loading and concatenating data options
### Project name and data format
