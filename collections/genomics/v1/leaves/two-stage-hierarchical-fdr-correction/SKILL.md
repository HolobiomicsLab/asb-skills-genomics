---
name: two-stage-hierarchical-fdr-correction
description: Use when you have pseudobulk per cell-type isoform counts (e.g. from
  scywalker's pseudobulk module) for two cell groups and need genes with significant
  differential transcript usage at gene- and transcript-level FDR control.
when_to_use_negative:
- Input is per-cell or bulk gene-level (not isoform/transcript-level) counts — DTU
  testing requires multi-isoform genes; use differential expression instead
- More than two group levels with a simple two-group design — the described workflow
  uses a two-group comparison design matrix and would need redesign
- Counts come from short-read data or non-scywalker pseudo-bulk files with a different
  isoform id schema that lacks transcript-to-gene mapping required for the two-stage
  split
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: Produces the pseudobulk (per cell-type) isoform counts and group assignments
    that feed the DTU test
  repo: https://github.com/derijkp/scywalker
- name: DRIMSeq
  role: Dirichlet-multinomial gene-level DTU screening and transcript-level testing
    of the pseudobulk isoform counts
- name: stageR
  role: Hierarchical two-stage correction yielding gene-level gFDR and transcript-level
    tFDR
- name: scsorter
  role: Assigns cell types used to define the compared cell groups when a -cellmarkerfile
    is given
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:a21e3d18e566883110d4962c4377b46fe05a0b1eeb65bf5cf70bc03677537862
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - DRIMSeq
  - scsorter
  - scywalker
  - stageR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# two-stage-hierarchical-fdr-correction

## Summary

Applies DRIMSeq's Dirichlet-multinomial gene-level screening followed by stageR hierarchical two-stage FDR correction to identify genes with differential transcript usage (gDTU) between cell groups and assign per-transcript tFDR values from scywalker pseudobulk isoform counts.

## When to use

Use when you have pseudobulk per cell-type isoform counts (e.g. from scywalker's pseudobulk module) for two cell groups and need genes with significant differential transcript usage at gene- and transcript-level FDR control.

## When NOT to use

- Input is per-cell or bulk gene-level (not isoform/transcript-level) counts — DTU testing requires multi-isoform genes; use differential expression instead
- More than two group levels with a simple two-group design — the described workflow uses a two-group comparison design matrix and would need redesign
- Counts come from short-read data or non-scywalker pseudo-bulk files with a different isoform id schema that lacks transcript-to-gene mapping required for the two-stage split

## Inputs

- scywalker pseudobulk (per cell-type) isoform count matrix (produced when -cellmarkerfile is supplied so scsorter assigns cell types)
- group/cell-type assignment table defining the two groups to compare (e.g. neuronal vs glial brain samples)

## Outputs

- dtu_significant_genes.tsv with significant gDTU genes and their gene-level gFDR and per-transcript tFDR values at FDR < 0.05

## How to apply

Load the pseudobulk (per cell-type) isoform count matrix produced by scywalker's celltyping/pseudobulk module (generated when a -cellmarkerfile triggers scsorter), together with the group/cell-type assignments defining the two groups to compare (e.g. neuronal vs glial brain cell types). Filter the count table to genes and transcripts with sufficient expression for DTU testing and build a two-group design matrix. Screen genes with DRIMSeq's Dirichlet-multinomial model to obtain gene-level gDTU candidate p-values, then test individual transcripts within screened genes with DRIMSeq. Apply stageR's hierarchical two-stage correction to obtain gene-level gFDR and transcript-level tFDR, retain results at FDR < 0.05, and write significant genes with gFDR and per-transcript tFDR to dtu_significant_genes.tsv. This two-stage procedure mirrors the paper's reference result of 301 gDTU genes (FDR < 0.05) between neuronal and glial brain cell types, including neuron-specific isoforms such as SEPTIN8 and novel differentially used isoforms in genes like PNKD.

## Related tools

- **scywalker** (Produces the pseudobulk (per cell-type) isoform counts and group assignments that feed the DTU test) — https://github.com/derijkp/scywalker
- **DRIMSeq** (Dirichlet-multinomial gene-level DTU screening and transcript-level testing of the pseudobulk isoform counts)
- **stageR** (Hierarchical two-stage correction yielding gene-level gFDR and transcript-level tFDR)
- **scsorter** (Assigns cell types used to define the compared cell groups when a -cellmarkerfile is given)

## Examples

```
Rscript -e 'library(DRIMSeq); library(stageR); st <- stageR(pScreen=gene_p, pConfirmation=tx_p, pScreenAdjusted=FALSE); st <- stageWiseAdjustment(st, method="dtu", alpha=0.05); sig <- getAdjustedPValues(st); write.table(sig, "dtu_significant_genes.tsv", sep="\t")'
```

## Evaluation signals

- Every gene in dtu_significant_genes.tsv passes FDR < 0.05 on both gFDR and tFDR
- Per-transcript tFDR values are present for all transcripts reported within each significant gene
- Result scale is consistent with the paper's brain-sample reference of 301 gDTU genes at FDR < 0.05
- Known biology is recapitulated: neuron-specific isoforms such as SEPTIN8 appear among significant genes
- Only genes that passed the DRIMSeq Dirichlet-multinomial screening stage enter the stageR confirmation stage

## Limitations

- Genes and transcripts with insufficient expression must be filtered out before testing; low-expression isoforms cannot be assessed
- Reference scale (301 gDTU genes) is specific to the paper's brain samples; other tissues will differ in result magnitude
- The two-stage procedure is tied to a two-group comparison design; it is not directly a multi-group test

## Evidence

- [other] Screen genes for differential transcript usage with DRIMSeq's Dirichlet-multinomial model to identify candidate gDTU genes.: [evidence span withheld — anchor status: not_found]
- [other] Test individual transcripts within the screened genes with DRIMSeq and apply stageR's hierarchical two-stage correction to obtain gene-level gFDR and transcript-level tFDR.: [evidence span withheld — anchor status: not_found]
- [other] Retain significant results at FDR < 0.05 and write a table of significant genes with their gFDR and per-transcript tFDR to dtu_significant_genes.tsv: [evidence span withheld — anchor status: not_found]
- [readme] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample"
