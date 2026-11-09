---
name: single-cell-celltype-group-comparison
description: Use when pseudobulk per-cell-type isoform counts from long-read single-cell
  data are available for two cell groups and the goal is to find genes with differential
  transcript usage (gDTU) and per-transcript significance.
when_to_use_negative:
- No cell-type markerset was supplied, so scywalker did not run scsorter and no pseudobulk
  per-cell-type counts exist to compare
- The goal is gene-level differential expression rather than differential transcript/isoform
  usage (DTU)
- More than two groups must be contrasted simultaneously without a pairwise design,
  since the procedure is built around a two-group comparison design matrix
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: Generates the pseudobulk per-cell-type isoform count matrix and comparison
    count files that feed the DTU analysis
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Assigns cell types from the provided markerset so pseudobulk (per cell-type)
    counts can be built
- name: sctype
  role: Alternative cell-typing tool that can use its internal database with the given
    tissue
- name: DRIMSeq
  role: Dirichlet-multinomial screening of genes for gDTU and transcript-level testing
    within screened genes
- name: stageR
  role: Hierarchical two-stage correction yielding gene-level gFDR and transcript-level
    tFDR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:12b182e36b0c22f8811c543e80eecf7aff1813a9547ecbe66637bb83e1168005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - DRIMSeq
  - scsorter
  - sctype
  - scywalker
  - stageR
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-celltype-group-comparison

## Summary

Compares assigned cell-type groups (e.g. neuronal vs glial) using scywalker pseudobulk isoform counts and a two-stage DRIMSeq/stageR Dirichlet-multinomial DTU analysis, yielding genes with differential transcript usage (gDTU) at FDR < 0.05 with per-transcript tFDR values.

## When to use

Use when pseudobulk per-cell-type isoform counts from long-read single-cell data are available for two cell groups and the goal is to find genes with differential transcript usage (gDTU) and per-transcript significance.

## When NOT to use

- No cell-type markerset was supplied, so scywalker did not run scsorter and no pseudobulk per-cell-type counts exist to compare
- The goal is gene-level differential expression rather than differential transcript/isoform usage (DTU)
- More than two groups must be contrasted simultaneously without a pairwise design, since the procedure is built around a two-group comparison design matrix

## Inputs

- scywalker pseudobulk (per cell-type) isoform count matrix from the celltyping/pseudobulk module (requires a -cellmarkerfile so scsorter runs)
- Group/cell-type assignments defining the two cell groups to be compared (e.g. group_filtered assignments from scsorter/sctype)

## Outputs

- dtu_significant_genes.tsv — table of significant gDTU genes with gene-level gFDR and per-transcript tFDR values
- Set of genes with differential transcript usage between the two cell groups at FDR < 0.05

## How to apply

Load the pseudobulk (per cell-type) isoform count matrix produced by scywalker's celltyping/pseudobulk module (generated when a -cellmarkerfile is supplied so scsorter runs), together with the group/cell-type assignments defining the two groups to compare. Filter the isoform count table to genes and transcripts with sufficient expression for DTU testing and set up a two-group comparison design matrix. Screen genes for differential transcript usage using DRIMSeq's Dirichlet-multinomial model, then test individual transcripts within screened genes and apply stageR's hierarchical two-stage correction to obtain gene-level gFDR and transcript-level tFDR. Retain significant results at FDR < 0.05, mirroring the paper's procedure that identified 301 gDTU genes between neuronal and glial brain cell types, including neuron-specific isoforms such as SEPTIN8 and novel differentially used isoforms in genes like PNKD.

## Related tools

- **scywalker** (Generates the pseudobulk per-cell-type isoform count matrix and comparison count files that feed the DTU analysis) — https://github.com/derijkp/scywalker
- **scsorter** (Assigns cell types from the provided markerset so pseudobulk (per cell-type) counts can be built)
- **sctype** (Alternative cell-typing tool that can use its internal database with the given tissue)
- **DRIMSeq** (Dirichlet-multinomial screening of genes for gDTU and transcript-level testing within screened genes)
- **stageR** (Hierarchical two-stage correction yielding gene-level gFDR and transcript-level tFDR)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir refdir -cellmarkerfile markers.tsv -threads 6 sample10x; Rscript dtu_drimseq_stager.R --counts sample10x/pseudobulk_iso_counts.tsv --groups neuronal,glial --fdr 0.05 --out dtu_significant_genes.tsv
```

## Evaluation signals

- Every gene in dtu_significant_genes.tsv passes the FDR < 0.05 threshold on both gFDR and tFDR
- The number of significant gDTU genes is on a scale consistent with the paper's brain-sample reference of 301 gDTU genes
- Known cell-type-specific isoforms (e.g. neuron-specific SEPTIN8) appear among the significant hits as a biological sanity check
- The pseudobulk input counts derive from emptydrops-approved cells (is_cell = 1) with filtered group assignments where available
- The design matrix correctly encodes the two cell groups being contrasted (e.g. neuronal vs glial)

## Limitations

- Cell typing is only as good as the supplied markerset; on limited data (e.g. the chromosome-17 test dataset) celltypes cannot be properly determined
- Isoform counts rely on scywalker's counting modes; strict counts require unique reads covering >= 90% of the transcript, so low-coverage transcripts may be dropped before DTU testing
- sctype falls back on its internal tissue database, which may not match the sample's biology

## Evidence

- [readme] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample"
- [readme] make count files allowing comparison between samples: "and make count files allowing comparison between samples for these pseudobulk counts."
- [results] counts_strict requiring >= 90% transcript coverage: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [methods] sctype internal database with given tissue: "however sctype will use its internal database 
    with the given tissue"
- [abstract] scywalker facilitates cell-type-specific novel isoform identification: "At the isoform level, we show that scywalker facilitates the direct identification of cell-type-specific expression of novel isoforms."
