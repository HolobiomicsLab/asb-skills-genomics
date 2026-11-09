---
name: cross-platform-gene-count-comparison
description: Use when two sequencing platforms have profiled the identical single-cell
  cDNA samples and you need to demonstrate concordance of cell-barcode-level or gene-level
  UMI count matrices via Pearson correlation.
when_to_use_negative:
- Input samples were not split and run on both platforms — comparing different cDNA
  samples would confound platform effects with sample effects
- The goal is isoform-level concordance rather than gene/UMI-level counts; the matrices
  compared here aggregate UMI counts, not isoform classifications
- Count matrices are not matched on shared cell barcodes or gene identifiers between
  platforms
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISA-Tools
  role: Provides the scGene_matrix utility that produces the single-cell gene/UMI
    expression quantities from HIT-scISOseq data used in the cross-platform correlation
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: R (Seurat/Matrix)
  role: Computes Pearson correlations between UMI count matrices and writes fig3_pearson_correlations.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:feedd1cf7a483d46056f632d3dd404368fb4102edfb64ef59ac71e664f3ea282
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - R
  - R (Seurat/Matrix)
  - Seurat
  - scISA-Tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-platform-gene-count-comparison

## Summary

Quantify agreement between HIT-scISOseq (PacBio Sequel II CCS) and Illumina short-read NGS measurements of the same single-cell 10x Genomics corneal limbal cDNA samples by computing Pearson correlations of UMI counts aggregated per cell barcode, per gene, and between biological replicates.

## When to use

Use when two sequencing platforms have profiled the identical single-cell cDNA samples and you need to demonstrate concordance of cell-barcode-level or gene-level UMI count matrices via Pearson correlation.

## When NOT to use

- Input samples were not split and run on both platforms — comparing different cDNA samples would confound platform effects with sample effects
- The goal is isoform-level concordance rather than gene/UMI-level counts; the matrices compared here aggregate UMI counts, not isoform classifications
- Count matrices are not matched on shared cell barcodes or gene identifiers between platforms

## Inputs

- Cell-by-barcode UMI count matrices from HIT-scISOseq and Illumina NGS (Source Data for Fig. 3)
- Cell-by-gene UMI count matrices from HIT-scISOseq and Illumina short-read NGS
- UMI count matrices from two HIT-scISOseq biological replicates (same limbal epithelium cDNA samples)

## Outputs

- fig3_pearson_correlations.csv containing Pearson r per comparison label (cellBC-level, gene-level, replicate-level)
- Validation report of each recomputed Pearson r versus reported values 0.992, 0.956, and 0.998 (tolerance ±0.01)

## How to apply

Load the deposited cell-by-BC, cell-by-gene, and replicate UMI count matrices from the Source Data for Fig. 3 into R (R-3.4.1 or later, with Seurat/Matrix available per the repo dependencies). Sum UMI counts per cell barcode and per gene using the scGene_matrix output of scISA-Tools for the HIT-scISOseq side and the Illumina short-read counts for the NGS side. Compute Pearson correlations between platforms on UMI totals by cellBC (expected r=0.992) and by gene (expected r=0.956), and between the two HIT-scISOseq biological replicates (expected r=0.998). Write the three coefficients with their comparison labels to fig3_pearson_correlations.csv and accept the result only if each recomputed r falls within ±0.01 of the reported values.

## Related tools

- **scISA-Tools** (Provides the scGene_matrix utility that produces the single-cell gene/UMI expression quantities from HIT-scISOseq data used in the cross-platform correlation) — https://github.com/shizhuoxing/scISA-Tools
- **R (Seurat/Matrix)** (Computes Pearson correlations between UMI count matrices and writes fig3_pearson_correlations.csv)

## Examples

```
Rscript -e 'm <- readRDS("fig3_matrices.rds"); r <- cor(Matrix::rowSums(m$hit_gene), m$ngs_gene, method="pearson"); write.csv(data.frame(comparison="gene", r=r), "fig3_pearson_correlations.csv")'
```

## Evaluation signals

- Pearson r for UMI counts by cellBC must fall within ±0.01 of the reported 0.992
- Pearson r for UMI counts by gene must fall within ±0.01 of the reported 0.956
- Pearson r between the two HIT-scISOseq biological replicates must fall within ±0.01 of the reported 0.998
- fig3_pearson_correlations.csv must contain all three comparison labels with their coefficients
- Correlations must be computed on shared cell barcodes/genes present in both platforms' matrices

## Limitations

- Only two biological replicates (one sample each from two animals) with no randomization or blinding, limiting statistical generalizability
- Concordance is claimed only on the same 10x Genomics limbal cDNA samples; compatibility with other platforms is asserted but not demonstrated
- Reproducibility depends on deposited Source Data matrices; the underlying wet-lab protocol sits behind an access-restricted protocols.io URL

## Evidence

- [results] Strong HIT-scISOseq/Illumina NGS concordance: [evidence span withheld — anchor status: not_found]
- [methods] Gene matrix generation tool: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
- [discussion] Illumina sequencing platform: "Illumina libraries were sequenced on a NextSeq 550 system (SY-415-1002, Illumina) by using NextSeq High Output Kits (150 cycles; 20024907, Illumina)"
- [readme] R environment dependency: "R-3.4.1 or later with `ggplot2 | gridExtra | grid | Seurat | DropletUtils`"
- [full_text] Replicate concordance target: [evidence span withheld — anchor status: not_found]
