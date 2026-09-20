---
name: sample-level-transcript-annotation-merging
description: Use when converting aligned Iso-Seq SAM/BAM alignments (from uLTRA or
  minimap2 mapping of FLNC reads) into a non-redundant per-sample genome annotation
  in BED12 format.
when_to_use_negative:
- Input reads have not yet been mapped to the genome — the pipeline stage expects
  SAM alignments from uLTRA/minimap2, not raw FLNC FASTA/BAM
- The goal is a multi-sample consensus annotation rather than a per-sample one — TAMA
  merge in this workflow combines chunks within a sample, not across samples
- Only primer/chimera/polyA cleanup is needed (LIMA, isoseq3 refine, TAMA polyA cleanup)
  — those are upstream filtering steps, not annotation collapsing
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: nf-core/isoseq
  role: Pipeline that orchestrates the collapse/merge annotation stage on aligned
    Iso-Seq reads
  repo: https://github.com/nf-core/isoseq
- name: TAMA
  role: Provides TAMA collapse (transcript model collapsing) and TAMA merge (merging
    chunk-level BED12 annotations per sample)
  repo: https://github.com/GenomeRIK/tama
- name: samtools
  role: Sorts alignments by genomic coordinates and converts SAM to BAM before collapsing
- name: uLTRA
  role: One of the splice-aware aligners whose SAM output feeds the collapse/merge
    stage
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: Alternative splice-aware aligner producing the SAM input to the collapse/merge
    stage
  repo: https://github.com/lh3/minimap2
- name: Nextflow
  role: Workflow engine in which the nf-core/isoseq collapse/merge modules are executed
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:d21f1ef4e91490e48cdaf817ef61ac9db226de5794fa8dcd693d0d3b8c80ea09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/isoseq
  tools:
  - Nextflow
  - TAMA
  - minimap2
  - nf-core/isoseq
  - samtools
  - uLTRA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sample-level-transcript-annotation-merging

## Summary

Collapse splice-consistent long-read alignments into non-redundant transcript models with TAMA collapse, then merge chunk-level collapsed BED12 annotations into a single per-sample annotation using TAMA merge, as done in the nf-core/isoseq pipeline after uLTRA/minimap2 genome mapping.

## When to use

Use when converting aligned Iso-Seq SAM/BAM alignments (from uLTRA or minimap2 mapping of FLNC reads) into a non-redundant per-sample genome annotation in BED12 format.

## When NOT to use

- Input reads have not yet been mapped to the genome — the pipeline stage expects SAM alignments from uLTRA/minimap2, not raw FLNC FASTA/BAM
- The goal is a multi-sample consensus annotation rather than a per-sample one — TAMA merge in this workflow combines chunks within a sample, not across samples
- Only primer/chimera/polyA cleanup is needed (LIMA, isoseq3 refine, TAMA polyA cleanup) — those are upstream filtering steps, not annotation collapsing

## Inputs

- SAM alignment files from uLTRA or minimap2 genome mapping of FLNC reads (per sample, possibly chunked)
- Reference genome to which reads were aligned

## Outputs

- Sorted BAM files (samtools sort output per alignment chunk)
- <sample>_collapsed.bed files (TAMA collapse transcript models per chunk)
- <sample>.bed — merged non-redundant per-sample annotation in 12-column BED12 format

## How to apply

For each sample, take the SAM alignment files produced by the uLTRA or minimap2 genome-mapping step and sort them by genomic coordinates while converting SAM to BAM with samtools sort. Collapse similar splice-consistent alignments into non-redundant transcript models with TAMA collapse, producing <sample>_collapsed.bed files for each chunk of alignments. Then combine the chunk-level collapsed BED12 annotations for the same sample into one common per-sample annotation using TAMA merge, writing the result as <sample>.bed. The rationale is that collapsing removes redundancy among reads sharing splice junctions, while merging unifies annotations generated from alignment chunks so each sample yields a single coherent transcript model set. Validate that the merged <sample>.bed is a valid 12-column BED12 file with intact block/exon structure.

## Related tools

- **nf-core/isoseq** (Pipeline that orchestrates the collapse/merge annotation stage on aligned Iso-Seq reads) — https://github.com/nf-core/isoseq
- **TAMA** (Provides TAMA collapse (transcript model collapsing) and TAMA merge (merging chunk-level BED12 annotations per sample)) — https://github.com/GenomeRIK/tama
- **samtools** (Sorts alignments by genomic coordinates and converts SAM to BAM before collapsing)
- **uLTRA** (One of the splice-aware aligners whose SAM output feeds the collapse/merge stage) — https://github.com/ksahlin/ultra
- **minimap2** (Alternative splice-aware aligner producing the SAM input to the collapse/merge stage) — https://github.com/lh3/minimap2
- **Nextflow** (Workflow engine in which the nf-core/isoseq collapse/merge modules are executed) — https://github.com/nf-core/isoseq

## Examples

```
nextflow run nf-core/isoseq --input samplesheet.csv --genome GRCh38.fa --aligner minimap2 -profile docker
```

## Evaluation signals

- Each sorted alignment chunk is a valid BAM file (samtools sort completed without error)
- TAMA collapse produced a <sample>_collapsed.bed file for every alignment chunk
- The final <sample>.bed is a valid 12-column BED12 file
- Block/exon structure fields in the merged BED12 are intact and internally consistent (block counts match block sizes/start entries)
- The merged annotation contains no duplicate/redundant transcript models relative to the per-chunk collapsed inputs

## Limitations

- No software version numbers are recorded for the tools in this changelog section, creating a reproducibility gap for pinning TAMA/samtools versions
- uLTRA can produce spurious alignments when a malformed GTF is used; removing the bioperl module stops the pipeline in that case, leaving malformed-GTF handling unaddressed
- Later pipeline revisions removed the samtools sort module ('samtools sort module is not needed anymore and has been removed'), so this exact step order may not hold for all pipeline versions

## Evidence

- [other] The post-processing stage sorts the SAM alignments into BAM (samtools sort), then uses TAMA collapse to collapse similar alignments into transcript models: [evidence span withheld — anchor status: not_found]
- [other] TAMA merge to combine multiple chunk-level annotations into one common per-sample annotation, which is emitted as a bed12 <sample>.bed file: [evidence span withheld — anchor status: not_found]
- [abstract] nf-core/isoseq is a pipeline for automatic read processing and genome annotation from Iso-Seq data: [evidence span withheld — anchor status: not_found]
- [intro] SAMTOOLS SORT - Sort alignment and convert sam file into bam file: "- [SAMTOOLS SORT](#samtools-sort) - Sort alignment and convert sam file into bam file"
- [readme] This software was designed for processing Iso-Seq data and other long read transcriptome data: [evidence span withheld — anchor status: external_doc]
