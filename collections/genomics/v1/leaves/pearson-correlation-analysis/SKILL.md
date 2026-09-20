---
name: pearson-correlation-analysis
description: Use when two count matrices measured on the same 10x Genomics single-cell
  cDNA samples (e.g., cellBC-level or gene-level UMI counts from HIT-scISOseq vs Illumina
  NGS, or two biological replicates) must be compared for agreement.
when_to_use_negative:
- The input matrices come from different cells/samples without shared cell barcodes
  or gene identifiers, so paired correlation is undefined
- The goal is isoform-level (not gene/UMI-level) comparison, which requires the scIsoform_matrix
  output and SQANTI3-filtered isoform set rather than UMI totals
- Raw CCS/FLNC read data still needs demultiplexing — run scISA-Tools (classify_by_primer,
  cellBC_UMI_corrector) before any correlation
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: scISA-Tools
  role: Demultiplexes HIT-scISOseq concatenated reads and produces the single-cell
    gene/UMI count matrices (scGene_matrix) that are correlated against Illumina NGS
    counts
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: R
  role: Computes Pearson correlations between aggregated UMI count vectors and writes
    the results CSV (R-3.4.1 or later with Seurat/Matrix dependencies)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:de13c8445b08e491482aa8eb4c6f8f34cc2fd867d446a5c12ed26739b423f174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - R
  - Seurat
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pearson-correlation-analysis

## Summary

Quantify concordance between HIT-scISOseq (PacBio Sequel II long-read) and Illumina short-read NGS single-cell UMI/gene count matrices, and between HIT-scISOseq biological replicates, by computing Pearson correlation coefficients on aggregated UMI counts.

## When to use

Use when two count matrices measured on the same 10x Genomics single-cell cDNA samples (e.g., cellBC-level or gene-level UMI counts from HIT-scISOseq vs Illumina NGS, or two biological replicates) must be compared for agreement.

## When NOT to use

- The input matrices come from different cells/samples without shared cell barcodes or gene identifiers, so paired correlation is undefined
- The goal is isoform-level (not gene/UMI-level) comparison, which requires the scIsoform_matrix output and SQANTI3-filtered isoform set rather than UMI totals
- Raw CCS/FLNC read data still needs demultiplexing — run scISA-Tools (classify_by_primer, cellBC_UMI_corrector) before any correlation

## Inputs

- Cell-by-barcode UMI count matrix (HIT-scISOseq, from Source Data for Fig. 3)
- Cell-by-gene UMI count matrix (HIT-scISOseq, from Source Data for Fig. 3)
- Illumina NGS short-read gene-level UMI count matrix for the same limbal epithelium cDNA samples
- UMI count matrices from two HIT-scISOseq biological replicates

## Outputs

- fig3_pearson_correlations.csv containing three labeled Pearson correlation coefficients (0.992 cellBC-level, 0.956 gene-level, 0.998 replicate-level)

## How to apply

Load the deposited single-cell gene/UMI count matrices from the Source Data for Fig. 3 (cell-by-BC, cell-by-gene, and replicate matrices) into R (R-3.4.1 or later with Matrix/Seurat available per the scISA-Tools dependencies). Sum UMI counts per cell barcode and compute the Pearson correlation between UMI totals and cellBC-based counts (expected r=0.992); aggregate UMI counts by gene using the scGene_matrix-style gene-level output and correlate HIT-scISOseq gene-level UMI counts against Illumina NGS short-read counts (expected r=0.956); finally correlate the UMI count matrices of the two HIT-scISOseq biological replicates (expected r=0.998). The rationale is that Pearson correlation on identically barcoded samples directly tests whether the long-read demultiplexing pipeline (scISA-Tools, with cellBC correction at posterior probability >0.975) preserves quantitative expression measurements relative to the short-read gold standard. Write the three coefficients with comparison labels to fig3_pearson_correlations.csv.

## Related tools

- **scISA-Tools** (Demultiplexes HIT-scISOseq concatenated reads and produces the single-cell gene/UMI count matrices (scGene_matrix) that are correlated against Illumina NGS counts) — https://github.com/shizhuoxing/scISA-Tools
- **R** (Computes Pearson correlations between aggregated UMI count vectors and writes the results CSV (R-3.4.1 or later with Seurat/Matrix dependencies))

## Examples

```
Rscript -e 'd<-read.csv("fig3_pearson_correlations.csv"); r<-cor(hit_umi_by_gene, ngs_umi_by_gene, method="pearson"); write.csv(data.frame(comparison="gene", pearson_r=r), "fig3_pearson_correlations.csv")'
```

## Evaluation signals

- Recomputed Pearson r for UMI counts by cellBC falls within ±0.01 of the reported 0.992
- Recomputed Pearson r for gene-level UMI counts (HIT-scISOseq vs Illumina NGS) falls within ±0.01 of the reported 0.956
- Recomputed Pearson r between the two HIT-scISOseq biological replicates falls within ±0.01 of the reported 0.998
- fig3_pearson_correlations.csv contains exactly three labeled coefficient rows matching the three comparison types
- Input matrices have consistent cell barcode and gene identifier sets before aggregation (no unmatched cellBC/gene keys)

## Limitations

- Validation is limited to two biological replicates (one sample each), with no randomization or blinding reported in the study
- Correlations were only demonstrated for 10x Genomics corneal limbus cDNA samples; compatibility with other single-cell platforms is claimed but not demonstrated
- Downstream counts depend on upstream filters (UMI quality: no homopolymers, no N, no base quality <10; cellBC correction threshold >0.975; isoform matrix filter -minUMIcount 3 and detection in ≥5 cells), so violations propagate into the correlations

## Evidence

- [results] Pearson r = 0.992 for UMI counts by cellBC and r = 0.956 for UMI counts by gene: [evidence span withheld — anchor status: not_found]
- [abstract] scISA-Tools demultiplexing accuracy: "We also report the development of scISA-Tools that demultiplex HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity."
- [discussion] cellBC correction threshold: [evidence span withheld — anchor status: not_found]
- [readme] R dependencies for correlation analysis: "R-3.4.1 or later with `ggplot2 | gridExtra | grid | Seurat | DropletUtils`"
- [methods] gene-level matrix generation: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
