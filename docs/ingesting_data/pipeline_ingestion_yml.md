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

* <p class="parameter">resources</p><br>
    Computing resources to use, specifically the number of threads used for parallel jobs. Specified by the following three parameters:

    * <p class="parameter">threads_high</p><br>
  	    For each thread, there must be enough memory to load all your input files at once and create the MuData object. Defaults to 1.
  
    * <p class="parameter">threads_medium</p><br>
        For each thread, there must be enough memory to load your mudata and do computationally light tasks. Defaults to 1.

    * <p class="parameter">threads_low</p><br>
  	    For each thread, there must be enough memory to load text files and do plotting, requires much less memory than the other two. Defaults to 1.
        #TODO: Check if it really is for each thread

* <p class="parameter">condaenv</p><br>
    Path to conda environment to use for running panpipes, leave blank if running native or your cluster automatically inherits the login node environment

## Loading and concatenating data options
### Project name and data format

* <p class="parameter">project</p><br>
    Project name. Defaults to "test".

* <p class="parameter">sample_prefix</p><br>
    Prefix for sample names. Defaults to "test".
  	#TODO: Is this the right definition?

* <p class="parameter">use_existing_h5mu</p><br>
    If you have an existing MuData object (.h5mu file) that you want to run panpipes on, then store it in the folder where you intend to run the workflow, and call it ${sample_prefix}_unfilt.h5mu where ${sample_prefix} = sample_prefix argument above
	#TODO: specify which folder. The one of the workflow? What is the $ in the name?

* <p class="parameter">submission_file</p><br>
    Submission file format. For ingest, the submission file must contain the follwing columns: sample_id  rna_path  rna_filetype  (prot_path  prot_filetype tcr_path  tcr_filetype etc.)
  	An example submission file can be found at resources/sample_file_mm.txt
	#TODO: What is meant by format?

* <p class="parameter">metadatacols</p><br>
    Which metadata cols from the submission file do you want to include in the anndata object as a comma-separated string e.g. batch,disease,sex

* <p class="parameter">concat_join_type</p><br>
    Specifies which concat join is performed on the MuData objects. We recommended inner. See the [AnnData documentation](https://anndata.readthedocs.io/en/latest/concatenation.html#inner-and-outer-joins) for details. Defaults to inner.

### Modalities in the project

the qc scripts are independent and modalities are processed following this order. Set to True to abilitate modality(ies). 
Leave empty (None) or False to signal this modality is not in the experiment. By default, all modalities are set to False.

* <p class="parameter">modalities:</p><br>
    * <p class="parameter">rna</p><br>
    * <p class="parameter">prot</p><br>
    * <p class="parameter">bcr</p><br>
    * <p class="parameter">tcr</p><br>
    * <p class="parameter">atac</p><br>
    
### Integrating barcode level data
Integrating barcode level data, e.g. demultiplexing with hashtags, chemical tags or lipid tagging

if you have cell level metadata such as results from a demultiplexing algorithm, you can incorporate it into the mudata object, this should be stored in one csv containing 2 columns, barcode_id, and sample_id, which should match the sample_id column in the submission file.

* <p class="parameter">barcode_mtd</p><br>
  
    * <p class="parameter">include</p> Defaults to True.<br>
      
    * <p class="parameter">path</p><br> #TODO: What is meant by path?
      
    * <p class="parameter">metadatacols</p><br> #TODO: What is meant by metadatacols?
    

### Loading prot data - additional options
* <p class="parameter">protein_metadata_table</p><br>
    As default the ingest will choose the first column of the cellranger features.tsv.gz. To merge extra information about the antibodies e.g. whether they are hashing antibodies or isotypes. Create a table with the first column equivalent to the first column of cellrangers features.tsv.gz and specify it below (save as txt file)
    Make sure there are unique entries in this column for each prot. Include a column called isotype (containing True or False) if you want to qc isotypes. Include a column called hashing_ab (containing True or False) if your dataset contains hashing antibodies which you wish to split into a separate modality

* <p class="parameter">index_col_choice</p><br>
    If you want to update the mudata index with a column from protein_metadata_table, specify here
    If there are overlaps with the rna gene symbols, then you might have trouble down the line
    It is recommended to add something to make the labels unique e.g. "prot_CD4"

* <p class="parameter">load_prot_from_raw</p> Default: False<br>
    If providing separate prot and rna 10X outputs, then the pipeline will load the filtered rna 10X outputs and the raw prot 10X counts
    we assume in most cases that we want to treat filtered rna barcodes as the "real" cells
    and we want out prot assays barcodes to match rna.

* <p class="parameter">subset_prot_barcodes_to_rna</p><br>
    In which case set subset_prot_barcodes_to_rna: True (which is also the default setting) if explicitly set to False, the full prot matrix will be loaded into the mudata obect. Defaults to False.

## Quality Control (QC) options
### 10X cellranger files processing

* <p class="parameter">plot_10X_metrics</p> Default: True<br>
    if starting from cellranger outputs, you can parse multiple metrics_summary.csv files and plot them together
    the workflow looks for the metrics_summary file in the "outs" folder and will stop if it doesn't find it.
    Set this parameter to False if not using cellranger folders as input

### Doublets on RNA - Scrublet

* <p class="parameter">scr</p><br>
    The values here are the default values, if you were to leave a paramter pblank, it would default to these value,
  
    * <p class="parameter">expected_doublet_rate</p> Default: 0.06<br>
        the expected fraction of transcriptomes that are doublets, typically 0.05-0.1.
        Results are not particularly sensitive to this parameter")
    
    * <p class="parameter">sim_doublet_ratio</p> Default: 2<br>
        the number of doublets to simulate, relative to the number of observed transcriptomes.
        Setting too high is computationally expensive. Min tested 0.5

    * <p class="parameter">n_neighbours</p>Default: 20<br>
        Number of neighbors used to construct the KNN classifier of observed transcriptomes
        and simulated doublets.
        The default value of round(0.5*sqrt(n_cells)) generally works well.
    
    * <p class="parameter">min_counts</p>Default: 2<br>
        Used for gene filtering prior to PCA. Genes expressed at fewer than `min_counts` in fewer than `min_cells` (see below) are excluded"
    
    * <p class="parameter">min_cells</p>Default: 3<br>
        Used for gene filtering prior to PCA.
        Genes expressed at fewer than `min_counts` (see above) in fewer than `min_cells` are excluded.")
    
    * <p class="parameter">min_gene_variability_pctl</p>Default: 85<br>
        Used for gene filtering prior to PCA. Keep the most highly variable genes
        (in the top min_gene_variability_pctl percentile),
        as measured by the v-statistic [Klein et al., Cell 2015]")
    
    * <p class="parameter">n_prin_comps</p> Default: 30<br>
        Number of principal components used to embed the transcriptomes
        prior to k-nearest-neighbor graph construction
    
    * <p class="parameter">use_thr</p> Default: True<br>
        use a user defined thr to define min doublet score to split true from false doublets?
        if false just use what the software produces
        this threshold applies to plots, a=no actual fitlering takes place.
    
    * <p class="parameter">call_doublets_thr</p> Default: 0.25<br>
        if use_thr is True, this thr will be used to define doublets
    
    
### RNA QC    
this part of the pipeline allows to generate the QC parameters that will be used to 
evaluate inclusion/ exclusion criteria. Filtering of cells/genes happens in the next workflow (preprocess)
leave options blank to avoid running, "default" (to use the data stored within the package)

It's often practical to rely on known gene lists, for a series of tasks, like evaluating % of mitochondrial genes or
ribosomal genes, or excluding IGG genes from HVG selection. For the ingest workflow, 
We collect the cellcycle genes used in scanpy.score_genes_cell_cycle [Satija et al. (2015), Nature Biotechnology.] 
in in a file, panpipes/resources/cell_cicle_genes.tsv
and an example gene list in panpipes/resources/qc_genelist_1.0.csv 

| mod | feature| group  |
|-----| -------|--------|
| RNA | gene_1 | mt     |
| RNA | gene_2 | rp     |
| RNA | gene_1 | exclude|
| RNA | gene_1 | markerX|  

We define "actions" on them as follows:

specify the "group" name of the genes you want to use to apply the action i.e. calc_proportion: mt will calculate
proportion of reads mapping to the genes whose group is "mt"

(for pipeline_ingest.py)
calc_proportions: calculate proportion of reads mapping to X genes over total number of reads, per cell
score_genes: using scanpy.score_genes function, 

(for pipeline_preprocess.py)
exclude: exclude these genes from the HVG selection, if they are deemed HV.

### cell cycle action

ccgenes will plot the proportions of cell cycle genes (recommended to leave as default)

* <p class="parameter">ccgenes</p> Default: default<br>
    Setting the `ccgenes` to `default` will calculate the phase of the cell cycle in which the cell is by using 
    `scanpy.tl.score_genes_cell_cycle` using the file provided in panpipes/resources/cell_cicle_genes.tsv
    Users can create their own list and specify the path in the `ccgenes` param to score the cells with a custom list.
    If left blank, the cellcycle score will not be calculated.

### custom genes actions

* <p class="parameter">custom_genes_file</p> Default: resources/qc_genelist_1.0.csv<br>

* <p class="parameter">calc_proportions</p> Default: hb,mt,rp<br>

* <p class="parameter">score_genes</p> Default: MarkersNeutro<br>

### Plot QC
all metrics should be inputted as a comma separated string e.g. a,b,c

* <p class="parameter">plotqc_grouping_var</p> Default: orig.ident<br>
    base of the plots, it is a column in the obs, can be a comma separated string of multiple categorical 
    plotqc_grouping_var: sample_id,rna:channel,prot:sample
    normally is the channel also referred to "sample_id"
    if left blank, the base of the plot will be the `sample_id` of your submission file

### Plot RNA QC metrics
* <p class="parameter">plotqc_rna_metrics</p> Default: doublet_scores,pct_counts_mt,pct_counts_rp,pct_counts_hb,pct_counts_ig<br>
    other cell covariates in rna .obs

### Plot PROT QC metrics
requires prot_path to be included in the submission file
all metrics should be inputted as a comma separated string e.g. a,b,c

* <p class="parameter">plotqc_prot_metrics</p> Default: total_counts,log1p_total_counts,n_prot_by_counts,pct_counts_isotype
    as standard the following metrics are calculated for prot data
    per cell metrics:
    total_counts,log1p_total_counts,n_prot_by_counts,log1p_n_prot_by_counts
    if isotypes can be detected then the following are calculated also:
    total_counts_isotype,pct_counts_isotype
    choose which ones you want to plot here

* <p class="parameter">plot_metrics_per_prot</p> Default: total_counts,log1p_total_counts,n_cells_by_counts,mean_counts<br>
    Since the protein antibody panels usually count fewer features than the RNA, it may be interesting to
    visualize breakdowns of single proteins plots describing their count distribution and other qc options.
    choose which ones you want to plot here, for example
    n_cells_by_counts,mean_counts,log1p_mean_counts,pct_dropout_by_counts,total_counts,log1p_total_counts
    
* <p class="parameter">identify_isotype_outliers</p> Default: True<br>
    isotype outliers: one way to determine which cells are very sticky is to work out which cells have the most isotype UMIs
    associated to them, to label a cell as an isotype outlier, it must meet or exceed the following crietria:
    be in the above x% quantile by UMI counts, for at least n isotypes 
    (e.g. above 90% quantile UMIs in at least 2 isotypes)

* <p class="parameter">isotype_upper_quantile</p> Default: 90<br>
    
* <p class="parameter">isotype_n_pass</p> Default: 2<br>
    
### Profiling Protein Ambient background
It is useful to characterise the background of your gene expression assay and antibody binding assay.
Inspect the plots and decide if to apply corrections, for example see Cellbender or SoupX. (currently not included in panpipes)

PLEASE NOTE that this analysis can ONLY BE RUN IF YOU ARE PROVIDING RAW input starting from cellranger outputs
(so the "empty" droplets can be used to  estimate the BG )
setting asses_background to True when you don't have RAW inputs will stop the pipeline with an error.

* <p class="parameter">assess_background</p> Default: False
    Setting `assess_background` to True will:
    1. create h5mu from raw data inputs (expected as cellranger h5 or mtx folder, if you do not have this then set to False)
    2. Plot comparitive QC plots to compare distribution of UMI and feature counts in background and foreground
    3. Create heatmaps of the top features in the background, 
    so you can compare the background contamination per channel

* <p class="parameter">downsample_background</p> Default: True
    typically there is a lot of cells in the full raw cellranger outputs, 
    since we just want to get a picture of the background then we can subsample the data 
    to a more reasonable ize. If you want to keep all the raw data then set the following to False

### Files required for profiling ambient background or running dsb normalisation:
the raw_feature_bc_matrix folder from cellranger or equivalent.
The pipeline will automatically look for this as a .h5 or matrix folder 
if the {mod}_filetype is set to "cellranger" or "cellranger_multi" or 10X_h5 
based on the path specified in the submission file.

If you are using a different format of file input e.g. csv matrix, 
make sure the two files are named using the convention:
{file_prefix}_filtered.csv" and {file_prefix}_raw.csv

### Investigate per channel antibody staining
This can help determine any inconsistencies in staining per channel and other QC concerns.

* <p class="parameter">channel_col</p> Default: sample_id<br>
    If you want to run clr normalisation on a per channel basis, then you need to 
    specify which column in your submission file corresponds to the channel
    at the QC stage it can be useful to look at the normalised data on a per sample or channel basis 
    i.e. the 10X channel. 
    this is usually the sample_id column (otherwise leave the next parameter blank)

* <p class="parameter">save_norm_prot_mtx</p> Default: False<br>
    It is important to note that in ingest the per channel normalised prot scores are not saved in the mudata object
    this is because if you perform feature normalisation (clr normalisation margin 0 or dsb normalisation), 
    on subsets of cells then the normalised values cannot be simply concatenated. 
    The PROT normalisation is rerun pn the complete object in the preprocess pipeline 
    (or you can run this pipeline with channel_col set as None)
    it is important to note, if you choose to run the clr on a per channel basis, then it is not stored in the h5mu file.
    if you want to save the per channel normalised values set the following to True:

## PROT normalization

* <p class="parameter">normalisation_methods</p> Default: clr
    comma separated string of normalisation options
    options: dsb,clr 
    Setting normalization method to dsb without providing raw files will stop the pipeline with an error.
    more details in this vignette https://muon.readthedocs.io/en/latest/omics/citeseq.html
    dsb https://muon.readthedocs.io/en/latest/api/generated/muon.prot.pp.dsb.html
    clr https://muon.readthedocs.io/en/latest/api/generated/muon.prot.pp.clr.html

### CLR parameters

* <p class="parameter">clr_margin</p> Default: 0
    margin determines whether you normalise per cell (as you would RNA norm), 
    or by feature (recommended, due to the variable nature of prot assays). 
    CLR margin 0 is recommended for informative qc plots in this pipeline
    0 = normalise rowwise (per feature)
    1 = normalise colwise (per cell)

### DSB parameters

* <p class="parameter">quantile_clipping</p> Default: True
    quantile clipping, even with normalisation, 
    some cells get extreme outliers which can be clipped as discussed https://github.com/niaid/dsb
    maximum value will be set at the value of the 99.5% quantile, applied per feature
    note that this feature is in the default muon mu.pp.dsb code, but manually implemented in this code.

    in order to run DSB you must have access to the complete raw counts, including the empty droplets 
    from both rna and protein assays, 
    see details for how to make sure your files are compatible in the assess background section below

## Plot ATAC QC metrics 
we require initializing one csv file per aggregated ATAC/multiome experiment.
if you need to analyse multiple samples in the same project, aggregate them with the cellranger arc pipeline
for multiome samples we recommend, specifying the 10X h5 input "10x_h5"
per_barcode_metric is only avail on cellranger arc (multiome)

* <p class="parameter">is_paired</p> Default: True
    is this an ATAC alone or a multiome sample?

* <p class="parameter">partner_rna</p>
    this is NOT a multiome exp, but you have an RNA anndata that you would like to use for TSS enrichment, 
    leave empty if no rna provided

* <p class="parameter">features_tss</p>
    if this is a standalone atac (is_paired: False), please provide a feature file to run TSS enrichment.
    supported annotations for protein coding genes provided

* <p class="parameter">plotqc_atac_metrics</p> Default: n_genes_by_counts,total_counts,pct_fragments_in_peaks,atac_peak_region_fragments,atac_mitochondrial_reads,atac_TSS_fragments
    these will be used to plot and saved in the metadata
    all metrics should be inputted as a comma separated string e.g. a,b,c

## Plot Repertoire QC metrics
Repertoire data will be stored in one modality called "rep", if you provide both TCR and BCR data then this will be merged, 
but various functions will be fun on TCR and BCR separately
Review scirpy documentation for specifics of data storage https://scverse.org/scirpy/latest/index.html

* <p class="parameter">ir_dist</p>
    compute sequence distance metric (required for clonotype definition)
    for more info on the following args go to 
    https://scverse.org/scirpy/latest/generated/scirpy.pp.ir_dist.html#scirpy.pp.ir_dist
    leave blank for defaults

    * <p class="parameter">metric</p>
    
    * <p class="parameter">sequence</p>

* <p class="parameter">clonotype_definition</p>
    clonotype definition 
    for more info on the following args go to 
    https://scverse.org/scirpy/latest/generated/scirpy.tl.define_clonotypes.html#scirpy.tl.define_clonotypes
    leave blank for defaults

    * <p class="parameter">receptor_arms</p>
    
    * <p class="parameter">dual_ir</p>
    
    * <p class="parameter">within_group</p>

* <p class="parameter">plotqc_rep_metrics</p>
    Default:
  
        - is_cell
        - extra_chains
        - clonal_expansion
        - rep:receptor_type
        - rep:receptor_subtype
        - rep:chain_pairing
        - rep:multi_chain

    Available matrics:
    * rep:clone_id_size
    * rep:clonal_expansion
    * rep:receptor_type
    * rep:receptor_subtype
    * rep:chain_pairing
    * rep:multi_chain
    * rep:high_confidence
    * rep:is_cell
    * rep:extra_chains
