# Complement C3d enables cell-mediated immunity capable of distinguishing spontaneously transformed from non-transformed cells Analysis
Code and other plots from scRNA-seq and bulk RNA-seq analysis for the "C3d protein in multiple myeloma: segregating tumor immunity and autoimmunity" article

## scRNA-seq Analysis
The scRNA-seq analysis was done in two part:
	CellRanger: Pre-processing of FASTQ to counts
	Seurat: Processing of counts to downstream analyses shown in the article

 The code for the analysis can be found here: mgdmb_c3d_scrnaseq_analysis_final.Rmd

### CellRanger

We used the human MYC transgene in our mice model, so we had to create a custom genome reference to align the reads and create counts. We used GRCm38 as mouse reference and added the human MYC gene (ENSG00000136997) from GRCh38. The steps for this process is below (files will be named differently for own analysis):


1. find the MYC gene in GTF file in the human reference
```
grep ‘ENSG00000136997’ human_genes.gtf
```

2. extract gene entries from GTF file in human reference
```
grep ‘ENSG00000136997’ human_genes.gtf > human_MYC_genes.gtf
```

3. change seqname/chromosome entry in MYC GTF file
```
sed -i ‘s/chr8/chr8_human/’ human_MYC_genes.gtf
```

4. change gene name entry in MYC GTF file
```
sed -i 's/gene_name "MYC";/gene_name "hMYC";/' human_MYC_genes.gtf
```

5. find seqname/chromosome sequences in FASTA file in human reference (range)
```
grep ‘>chr8’ human_genome.fa
grep ‘>chr9’ human_genome.fa
```

6. extract sequences from FASTA file in human reference (not include ‘>chr9’ line which is last line)
```
sed -n '/>chr8/,/>chr9/p' human_genome.fa | sed '$d' > human_MYC_chr8.fa
```

7. change seqname/chromosome name to match GTF file in MYC FASTA file
```
sed -i ‘s/chr8/chr8_human/’ human_MYC_chr8.fa
```

8. combine FASTA files
```
cat mouse_genome.fa human_MYC_chr8.fa > new_genome.fa
```

9. combine GTF files
```
cat mouse_genes.gtf human_MYC_genes.gtf > new_genes.gtf
```

10. We then indexed the new custom reference:
```
cellranger mkref --genome=new_genome --fasta=new_genome.fa --genes=new_genes.gtf
```

11. Then we aligned the FASTQ files with this indexed reference in CellRanger. You can refer to the CellRanger manual for more information here:
(https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct)

### Seurat 

R code can be found in the “mgdmb_c3d_scrnaseq_analysis_final.Rmd” file

#### Input
The input files for this part are:
- Multiple_Myeloma_10X_mice_list_v3.xlsx = contain metadata for samples in dataset
- {*}_filtered_feature_bc_matrix.h5 = contains filtered count matrix from CellRanger; should have one for every sample
- {*}_filtered_contig_annotations.csv = contains filtered contigs (VDJ information) from CellRanger; should have one for every sample

The metadata file can be found in the ***tables_and_objects*** folder. The count matrices and contigs can be found in GEO. 

#### QC
Violin and scatter QC plots can be found in the ***plots/qc*** folder for before and after the filtering step.

#### Integration Analysis
We then ran the samples through Seurat's integration pipeline since we found batch effect from the “run_id”, which indicates that the samples were sequenced at different times. You can find the PCA and other QC plots in the ***plots/qc*** folder and the UMAP plots in the ***plot/umap*** folder. 

#### Plasma Cell Analysis
You can find the DE genes from a previous study in the ***gene_expression_in_MM_fig2a_data.xlsx*** file in the ***tables_and_objects*** folder. 

## Bulk RNA-seq Analysis

The code for the analysis can be found here: mgdmb_c3d_bulk_rnaseq_analysis_final.Rmd
