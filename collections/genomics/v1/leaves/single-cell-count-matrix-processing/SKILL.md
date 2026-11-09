---
name: single-cell-count-matrix-processing
description: Use when HIT-scISOseq PacBio HiFi reads (post cellBC/UMI correction)
  must be converted into cell-by-gene or cell-by-isoform count matrices, or when cross-platform
  agreement with 10x Genomics/NGS counts on the same cDNA needs to be measured.
when_to_use_negative:
- Input is raw CCS/subreads BAM that has not yet been classified by primer or barcode-corrected
  — scGene_matrix requires FLNC reads with valid cellBC/UMI
- The goal is isoform structural characterization or QC classification (SQANTI3/filter.tmap.pl
  territory), not building expression count matrices
- Data derive from platforms other than 10x Genomics — compatibility is claimed but
  not demonstrated in the study
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: scISA-Tools
  role: Provides scGene_matrix and scIsoform_matrix utilities to generate single-cell
    gene/isoform count matrices, plus cellBC_UMI_corrector for barcode/UMI correction
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: R (Seurat/Matrix)
  role: Loads the deposited count matrices, aggregates UMI totals per cellBC and per
    gene, and computes Pearson correlations (>=R-3.4.1 with ggplot2, gridExtra, Seurat,
    DropletUtils)
- name: CellRanger 3.1.0
  role: Supplies the whitelist barcode reference and python environment wrapped by
    cellBC_UMI_corrector
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:471eccd55f9c60435e5f17b37f6d7fe0fcefd2fa33724ecea5215d361caf6a07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - CellRanger 3.1.0
  - R
  - R (Seurat/Matrix)
  - Seurat
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-count-matrix-processing

## Summary

Build and validate single-cell UMI/gene count matrices from HIT-scISOseq long reads using the scISA-Tools scGene_matrix and scIsoform_matrix utilities, then quantify platform concordance against Illumina short-read NGS via Pearson correlations on cell-barcode and gene-aggregated UMI counts.

## When to use

Use when HIT-scISOseq PacBio HiFi reads (post cellBC/UMI correction) must be converted into cell-by-gene or cell-by-isoform count matrices, or when cross-platform agreement with 10x Genomics/NGS counts on the same cDNA needs to be measured.

## When NOT to use

- Input is raw CCS/subreads BAM that has not yet been classified by primer or barcode-corrected — scGene_matrix requires FLNC reads with valid cellBC/UMI
- The goal is isoform structural characterization or QC classification (SQANTI3/filter.tmap.pl territory), not building expression count matrices
- Data derive from platforms other than 10x Genomics — compatibility is claimed but not demonstrated in the study

## Inputs

- Corrected FLNC reads with cellBC/UMI (isoseq_flnc.BarcodeUMI.fastq) from classify_by_primer
- cellBC/UMI correction output from cellBC_UMI_corrector (CellRanger 3.1.0 whitelist 3M-february-2018.txt)
- Illumina short-read NGS count matrices from Source Data for Fig. 3 (cell-by-BC, cell-by-gene, replicate matrices)
- CellRanger-compatible cell barcode whitelist (3M-february-2018.txt.gz)

## Outputs

- Single-cell gene expression matrix per sample (scGene_matrix output)
- Single-cell isoform expression matrix per sample (scIsoform_matrix output, filtered -minUMIcount 3, >=5 cells)
- fig3_pearson_correlations.csv with UMI-by-cellBC (r=0.992), UMI-by-gene (r=0.956), and replicate (r=0.998) Pearson coefficients

## How to apply

After classify_by_primer and cellBC_UMI_corrector produce FLNC reads with corrected cell barcodes and UMIs, run the scGene_matrix utility to obtain single-cell gene expression quantities per sample and scIsoform_matrix to obtain isoform-level matrices; filter isoforms with -minUMIcount 3 and retain only isoforms detected in at least 5 cells. Sum UMI counts per cell barcode and per gene from the resulting matrices (e.g., in R with Seurat/Matrix on R-3.4.1 or later). To validate platform concordance on the same 10x Genomics corneal limbus cDNA, compute Pearson correlations of UMI totals by cellBC between HIT-scISOseq and Illumina NGS (expected r = 0.992), by gene (expected r = 0.956), and between the two HIT-scISOseq biological replicates (expected r = 0.998). Write the three coefficients with comparison labels to fig3_pearson_correlations.csv.

## Related tools

- **scISA-Tools** (Provides scGene_matrix and scIsoform_matrix utilities to generate single-cell gene/isoform count matrices, plus cellBC_UMI_corrector for barcode/UMI correction) — https://github.com/shizhuoxing/scISA-Tools
- **R (Seurat/Matrix)** (Loads the deposited count matrices, aggregates UMI totals per cellBC and per gene, and computes Pearson correlations (>=R-3.4.1 with ggplot2, gridExtra, Seurat, DropletUtils))
- **CellRanger 3.1.0** (Supplies the whitelist barcode reference and python environment wrapped by cellBC_UMI_corrector)

## Examples

```
Rscript -e 'm<-readRDS("hit_umi.rds"); out<-data.frame(comparison=c("cellBC","gene","replicate"), r=c(cor(h_cell,ngs_cell),cor(h_gene,ngs_gene),cor(rep1,rep2))); write.csv(out,"fig3_pearson_correlations.csv",row.names=FALSE)'
```

## Evaluation signals

- Pearson r for HIT-scISOseq vs Illumina NGS UMI counts by cellBC must be within ±0.01 of 0.992
- Pearson r for gene-level UMI counts vs NGS must be within ±0.01 of 0.956
- Pearson r between the two biological replicate UMI matrices must be within ±0.01 of 0.998
- Isoform matrix should exclude isoforms with UMI count <3 or detected in <5 cells across all samples
- Output CSV contains all three labeled correlation coefficients in fig3_pearson_correlations.csv

## Limitations

- CellBC correction accepts a whitelist barcode only when posterior probability exceeds 0.975; low-quality barcodes are discarded, affecting matrix sensitivity
- UMIs must pass quality filters (no homopolymers, no N bases, no bases with quality <10), which removes some reads before matrix construction
- The underlying UMI quality filter chain depends on CellRanger correction logic operating on long-read data alone, without short-read guidance

## Evidence

- [methods] scGene_matrix utility for gene-level matrices: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
- [methods] scIsoform_matrix utility for isoform matrices: "we are using `scIsoform_matrix` utility to get the single cell isoform expression quantity in each sample"
- [discussion] Isoform count matrix filter thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] CellBC correction posterior threshold: [evidence span withheld — anchor status: not_found]
- [readme] R dependencies for matrix processing: "R-3.4.1 or later with `ggplot2 | gridExtra | grid | Seurat | DropletUtils`"
