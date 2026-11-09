---
name: dirichlet-multinomial-dtu-gene-screening
description: Use when scywalker pseudobulk (per cell-type) isoform counts from long-read
  single-cell/nuclei data are available and the goal is to find genes with differential
  transcript usage between two assigned cell groups at FDR < 0.05.
when_to_use_negative:
- Input is bulk or single-sample isoform counts with no per-cell-type pseudobulk grouping
  — the design requires two assigned cell groups
- The analysis target is differential gene expression (total gene counts) rather than
  isoform/transcript usage proportions within genes
- No cellmarkerfile/markersets were supplied, so scsorter was not run and no pseudobulk
  per-cell-type counts exist
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: Produces the pseudobulk (per cell-type) isoform count matrices and cross-sample
    comparison count files that feed the DTU test
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Assigns cell types from the provided cellmarkerfile so pseudobulk per-cell-type
    isoform counts can be generated
- name: DRIMSeq
  role: Dirichlet-multinomial gene-level screening for gDTU genes and transcript-level
    testing within screened genes
- name: stageR
  role: Hierarchical two-stage FDR correction yielding gene-level gFDR and transcript-level
    tFDR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:534e58e589c17fe734f0bf30462144558dd189160a865b5b9db757699a5bfbad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - DRIMSeq
  - scsorter
  - scywalker
  - stageR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dirichlet-multinomial-dtu-gene-screening

## Summary

A two-stage differential transcript usage (DTU) analysis that screens genes with DRIMSeq's Dirichlet-multinomial model on scywalker pseudobulk per-cell-type isoform counts, then confirms individual transcripts with stageR's hierarchical FDR correction. It identifies genes switching isoform usage between cell groups (e.g. neuronal vs glial brain cells) with gene-level gFDR and transcript-level tFDR values.

## When to use

Use when scywalker pseudobulk (per cell-type) isoform counts from long-read single-cell/nuclei data are available and the goal is to find genes with differential transcript usage between two assigned cell groups at FDR < 0.05.

## When NOT to use

- Input is bulk or single-sample isoform counts with no per-cell-type pseudobulk grouping — the design requires two assigned cell groups
- The analysis target is differential gene expression (total gene counts) rather than isoform/transcript usage proportions within genes
- No cellmarkerfile/markersets were supplied, so scsorter was not run and no pseudobulk per-cell-type counts exist

## Inputs

- scywalker pseudobulk (per cell-type) isoform count matrix (e.g. counts_strict per-transcript counts per cell type)
- group/cell-type assignment table defining the two cell groups to be compared (e.g. neuronal vs glial, from scsorter/sctype with -cellmarkerfile)

## Outputs

- dtu_significant_genes.tsv: significant gDTU genes with gene-level gFDR and per-transcript tFDR values (FDR < 0.05)
- screened candidate gDTU gene list from the DRIMSeq Dirichlet-multinomial gene-level test

## How to apply

Load the pseudobulk (per cell-type) isoform count matrix produced by scywalker's celltyping/pseudobulk module (generated when a -cellmarkerfile is supplied so scsorter runs), together with the group/cell-type assignments defining the two groups to compare. Filter the isoform count table to genes and transcripts with sufficient expression for DTU testing and build a two-group comparison design matrix. Screen genes for differential transcript usage with DRIMSeq's Dirichlet-multinomial model to identify candidate gDTU genes, then test individual transcripts within those genes with DRIMSeq and apply stageR's hierarchical two-stage correction to obtain gene-level gFDR and per-transcript tFDR. Retain results at FDR < 0.05 and write significant genes with gFDR and tFDR to dtu_significant_genes.tsv, mirroring the paper's brain-sample analysis that yielded 301 gDTU genes.

## Related tools

- **scywalker** (Produces the pseudobulk (per cell-type) isoform count matrices and cross-sample comparison count files that feed the DTU test) — https://github.com/derijkp/scywalker
- **scsorter** (Assigns cell types from the provided cellmarkerfile so pseudobulk per-cell-type isoform counts can be generated)
- **DRIMSeq** (Dirichlet-multinomial gene-level screening for gDTU genes and transcript-level testing within screened genes)
- **stageR** (Hierarchical two-stage FDR correction yielding gene-level gFDR and transcript-level tFDR)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Every gene in dtu_significant_genes.tsv passes FDR < 0.05 on both gFDR and tFDR
- Result scale is consistent with the paper's brain-sample reference of 301 gDTU genes between neuronal and glial cell types
- Count matrix was filtered to genes/transcripts with sufficient expression before DRIMSeq testing, and a two-group design matrix was used
- Known neuron-specific isoforms (e.g. SEPTIN8) and novel differentially used isoforms (e.g. PNKD) are recovered among significant genes
- Pseudobulk input counts trace back to emptydrops-approved cells (is_cell flag) with cell types assigned via scsorter/sctype

## Limitations

- Cell typing and pseudobulk counts require cell specific markersets; without -cellmarkerfile, scsorter will not run and no per-cell-type counts are made
- scywalker's isoform counting distinguishes weighed, unique, and strict counts (strict requires >= 90% transcript coverage), so count definition affects DTU input
- On very limited test data (e.g. the chromosome 17 example), cell types cannot be properly determined, making DTU results unreliable

## Evidence

- [other] The two-stage DTU analysis of scywalker pseudobulk isoform counts identified 301 genes with significant differential transcript usage (FDR < 0.05) between neuronal and glial brain cell types: [evidence span withheld — anchor status: not_found]
- [intro] scywalker pseudobulk and cell-type assignment capability: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample"
- [results] isoform count strictness definition: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [readme] celltyping precondition for pseudobulk: "If cell specific markersets are provided, it will also assign cell-types
and generate pseudobulk (per cell-type) counts for each sample"
- [abstract] novel isoform cell-type specificity: "At the isoform level, we show that scywalker facilitates the direct identification of cell-type-specific expression of novel isoforms."
