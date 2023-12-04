# Ingesting complex data with panpipes

Experimental designs of single cell experiments are often complex, both due to the increasingly more sophisticated assays that allow many library types to be produced from one sample, the elaborate set-up needed to answer biological question (e.g. inclusion of multiple tissue collection sites and sample collection time points from a single patients), as well as challanges presented by single cell data - with libraries sometimes underperforming at the sequencing stage or even failing.

We will consider a hypothetical single cell experiment with 5 modalities:
- Gene expression data (*rna*)
- A panel of barcoded antibodies (adt included in *prot*)
- A panel of barcoded cell hashing oligos (hto included in *prot*)
- B cell receptor sequence (bcr included in *vdj*)
- T cell receptor sequence (tcr included in *vdj*)
This experiment is exemplifying some of the more complex design cases we encountered when using the pipeline.

Each well of the 10X Chromium chip would have been run with samples from one patient, but different conditions A and B (for example, two different tissues, tumour and healthy, or two different timepoints, pre- and post-treatment). The hashing oligos would have been different for each sample, to ensure that i) any sample mix-ups in the lab can be promptly detected and ii) well-to-well cross-contamination is minimised.

In addition, we will imagine that some libraries in the experiment were lost, not made (due to, for example, insufficient cDNA), or failed during sythesis.

So, our experimental design table with libraries `cellranger multi` outputs should be something like this:

| Patient | RNA | PROT | TCR | BCR |
| ------- | --- | ---- | --- | --- |
| Patient 1 | RNA1 | PROT1 | TCR1 | BCR1 |
| Patient 2 | RNA2 | PROT2 | _missing_ | BCR2 |
| Patient 3 | RNA3 | PROT3 | TCR3 | _missing_ |

At current stage, the data cannot be ingested into `panpipes`. We need to split the conditions (de-multiplex, NB this is different from `fastq` files demultiplexing after sequencing and is based on hashing oligos) in each well of the chip so we know which cells come from condition A and which from condition B. This can be done using exisitng tools - we recommend [hadge](https://github.com/theislab/hadge) or [cellhashR](https://github.com/BimberLab/cellhashR).

At the end of the demultiplexing process, you should have a `.csv` file with cell barcodes and their assignment, which you can pass as a parameter in the `pipeline.yaml`.

In addition to assigning conditions A and B to cells, we can also identify doublets as droplets where both hashing antibodies are present. This will aid the `scrublet` doublet scoring function by default implemented in panpipes.

Your csv file should look something like that:

```
hashing.csv

sample_id,barcode_id,classification,is_doublet
patient1,AAACCTGAGAGCCTAG-1,conditionA,False
patient1,AAACCTGAGAGTGAGA-1,conditionB,False
patient1,AAACCTGAGCCAACAG-1,doublet,True
patient2,AAACCTGAGCCGCCTA-1,conditionA,False
patient2,AAACCTGAGGACTGGT-1,conditionB,False
```
 
`panpipes` will also neeed to know which antibodies carry biological signal (i.e. are recognising protein epitopes important in your experiment) and which discern experimental conditions A and B (i.e. are recognising an abundant protein complex such as MHC (_BioLegend_'s approach) or nuclear lamina (_10X_'s approach)).

It is a good idea to include a prefix in the antibody names to easily distinguish them from the gene names. This must be a `.txt` not `.csv` file:

```
protein.txt

name	isotype	hashing_ab
ADT_CD8	False	False
ADT_CD4	False	False
ADT_CD19	False	False
ADT_CD14	False	False
ADT_CD16	False	False
ADT_HLA-DR	False	False

HTO_1	False	True
HTO_2	False	True
HTO_3	False	True
HTO_4	False	True
```

Finally, your csv with the libraries data. It should be a `.csv` file containing the raw data (in our case `cellranger multi` outputs and should look like this:

```
config.csv

sample_id	gex_path	gex_filetype	adt_path	adt_filetype	tcr_path	tcr_filetype	bcr_path	bcr_filetype
patient1	./multi/patient1/patient1/outs/	cellranger_multi	./multi/patient1/patient1/outs/	cellranger_multi	./multi/patient1/patient1/outs/multi/vdj_t/all_contig_annotations.csv	cellranger_vdj	./multi/patient1/patient1/outs/multi/vdj_b/all_contig_annotations.csv	cellranger_vdj
patient1	./multi/patient1/patient1/outs/	cellranger_multi	./multi/patient1/patient1/outs/	cellranger_multi	None	cellranger_vdj	./multi/patient1/patient1/outs/multi/vdj_b/all_contig_annotations.csv	cellranger_vdj
patient1	./multi/patient1/patient1/outs/	cellranger_multi	./multi/patient1/patient1/outs/	cellranger_multi	./multi/patient1/patient1/outs/multi/vdj_t/all_contig_annotations.csv	cellranger_vdj	None	cellranger_vdj
```
Or in tabular format (assuming `path = ./multi/patient_id/patient_id/`):

| sample_id	| gex_path |	gex_filetype |	adt_path	| adt_filetype |	tcr_path |	tcr_filetype |	bcr_path |	bcr_filetype | disease |
| ---	| --- |	--- |	---	| --- |	--- |	--- |	--- |	--- | --- |
| patient1	| path/outs/	| cellranger_multi	| path/outs/	| cellranger_multi |	path/outs/multi/vdj_t/all_contig_annotations.csv |	cellranger_vdj	| path/outs/multi/vdj_b/all_contig_annotations.csv |	cellranger_vdj | cancer |
| patient2	| path/outs/	| cellranger_multi	| path/outs/	| cellranger_multi |	None |	cellranger_vdj	| path/outs/multi/vdj_b/all_contig_annotations.csv |	cellranger_vdj | cancer |
| patient3	| path/outs/	| cellranger_multi	| path/outs/	| cellranger_multi |	path/outs/multi/vdj_t/all_contig_annotations.csv |	cellranger_vdj	| None |	cellranger_vdj | healthy |

We could also include additional columns, such as `sex`, `bmi`, `diagnosis`, etc. in this csv, they would be appended to `adata.obs`. Our `condition` (A or B) will also be appended to the `adata.obs` within each patient sample.

We remember that for each patient, we used a different set of antibodies, therefore, patient1 will have HTO_1 and HTO_2, but patient2 will have HTO_3 and HTO_4, and so on. Therefore, we cannot use the default inner join, [which will take the intersection of common features](https://anndata.readthedocs.io/en/latest/concatenation.html#inner-and-outer-joins). We must specify:
```
concat_join_type: outer
```
and when performing PCA on the protein data (if desired), we must take the maximum number of calculates PCs to be (number of **shared** features - 1).

Finally, we are ready to update the `pipeline.yaml`:
```
submission_file: kymab_config.tsv
metadatacols: disease
concat_join_type: outer

modalities:
  rna: True
  prot: True
  bcr: True
  tcr: True
  atac: False


barcode_mtd:
  include: true
  path: hashing.csv
  metadatacols: classification, is_doublet

protein_metadata_table: protein.txt
```