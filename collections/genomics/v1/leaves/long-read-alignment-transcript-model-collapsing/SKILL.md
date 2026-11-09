---
name: long-read-alignment-transcript-model-collapsing
description: Use when full-length non-chimeric (FLNC) Iso-Seq reads must be aligned
  to a reference genome and reduced to a non-redundant gene/isoform annotation in
  BED12 format.
when_to_use_negative:
- The input reads are short (Illumina) reads — minimap2 splice presets used here target
  long noisy reads such as PacBio Iso-Seq/HiFi and Nanopore cDNA/direct RNA.
- The sample annotation is already a collapsed BED12/GTF transcript model set — re-running
  collapse/merge would be redundant.
- Only a malformed GTF is available for the uLTRA path — uLTRA produces spurious alignments
  in this case, so minimap2 should be used instead.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Splice-aware alignment of PacBio Iso-Seq FLNC reads to the reference genome,
    producing SAM output
  repo: https://github.com/lh3/minimap2
- name: uLTRA
  role: Alternative splice-guided long-read aligner guided by a database of exon annotations,
    producing SAM alignments
  repo: https://github.com/ksahlin/ultra
- name: BioPerl
  role: Removes spurious uLTRA alignments (uLTRA path only) caused by malformed GTF
    input
  repo: https://bioperl.org/
- name: samtools
  role: Sorts SAM alignments by genomic coordinates and converts them to BAM before
    collapsing
- name: TAMA
  role: TAMA collapse reduces similar alignments to non-redundant transcript models;
    TAMA merge combines chunk-level collapsed BED12 annotations into one per-sample
    annotation
  repo: https://github.com/GenomeRIK/tama
- name: nf-core/isoseq
  role: Nextflow pipeline orchestrating the alignment, filtering, sorting, and collapse/merge
    steps end to end
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:b4bf6c0fa35ce4a3fdec29d4b7ad755fb4892c497ea322c59ca32222d9fa2593
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - BioPerl
  - TAMA
  - minimap2
  - nf-core/isoseq
  - samtools
  - uLTRA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-alignment-transcript-model-collapsing

## Summary

Maps PacBio Iso-Seq FLNC long reads onto a reference genome with a splice-aware long-read aligner (uLTRA or minimap2) and collapses redundant splice-consistent alignments into a non-redundant per-sample transcript annotation in BED12 using TAMA collapse and TAMA merge.

## When to use

Use when full-length non-chimeric (FLNC) Iso-Seq reads must be aligned to a reference genome and reduced to a non-redundant gene/isoform annotation in BED12 format.

## When NOT to use

- The input reads are short (Illumina) reads — minimap2 splice presets used here target long noisy reads such as PacBio Iso-Seq/HiFi and Nanopore cDNA/direct RNA.
- The sample annotation is already a collapsed BED12/GTF transcript model set — re-running collapse/merge would be redundant.
- Only a malformed GTF is available for the uLTRA path — uLTRA produces spurious alignments in this case, so minimap2 should be used instead.

## Inputs

- FLNC FASTA reads after TAMA polyA clean-up (possibly gzipped for the uLTRA path)
- Reference genome FASTA
- Exon/GTF annotation database (required for uLTRA splice-guided alignment)
- SAM alignment files from uLTRA or minimap2

## Outputs

- Coordinate-sorted BAM files (from samtools sort)
- Per-chunk collapsed transcript models `<sample>_collapsed.bed` (TAMA collapse)
- Merged per-sample non-redundant BED12 annotation `<sample>.bed` (TAMA merge)

## How to apply

Run the alignment inside nf-core/isoseq (Nextflow) by choosing uLTRA or minimap2 (`MINIMAP2` or `uLTRA` aligns reads on the genome) using minimap2 splice presets appropriate for PacBio Iso-Seq, e.g. `-ax splice:hq -uf`. For the uLTRA path, decompress FLNC FASTAs (GUNZIP) and use BioPerl to remove spurious alignments that uLTRA produces when a malformed GTF is supplied. Sort each SAM alignment by genomic coordinates and convert to BAM with samtools sort. Then collapse splice-consistent similar alignments into transcript models with TAMA collapse (producing per-chunk `<sample>_collapsed.bed` files), and combine the chunk-level collapsed BED12 annotations for the same sample with TAMA merge, writing `<sample>.bed`. Rationale: collapsing removes redundant isoforms from chunked alignments so the final annotation is one non-redundant per-sample transcript model set.

## Related tools

- **minimap2** (Splice-aware alignment of PacBio Iso-Seq FLNC reads to the reference genome, producing SAM output) — https://github.com/lh3/minimap2
- **uLTRA** (Alternative splice-guided long-read aligner guided by a database of exon annotations, producing SAM alignments) — https://github.com/ksahlin/ultra
- **BioPerl** (Removes spurious uLTRA alignments (uLTRA path only) caused by malformed GTF input) — https://bioperl.org/
- **samtools** (Sorts SAM alignments by genomic coordinates and converts them to BAM before collapsing)
- **TAMA** (TAMA collapse reduces similar alignments to non-redundant transcript models; TAMA merge combines chunk-level collapsed BED12 annotations into one per-sample annotation) — https://github.com/GenomeRIK/tama
- **nf-core/isoseq** (Nextflow pipeline orchestrating the alignment, filtering, sorting, and collapse/merge steps end to end) — https://github.com/nf-core/isoseq

## Examples

```
minimap2 -ax splice:hq -uf ref.fa pacbio-ccs.fq.gz > aln.sam
```

## Evaluation signals

- The final `<sample>.bed` is a valid 12-column BED12 file with intact block/exon structure.
- Alignment output is a coordinate-sorted, valid BAM file (samtools sort exit success and BAM header check).
- The merged annotation is non-redundant: no duplicate transcript models remain across chunk-level collapsed BEDs after TAMA merge.
- For the uLTRA path, the BioPerl filter has removed spurious alignments; no alignments derived from malformed GTF artifacts persist in the sorted BAM.
- Each sample/run in the samplesheet yields its own merged `<sample>.bed` with the expected sample-level parallelization.

## Limitations

- uLTRA produces spurious alignments if a malformed GTF is used; removing the BioPerl module stops the pipeline in that case, leaving malformed-GTF handling unaddressed.
- The `samtools sort` module has been removed in later versions of the pipeline (no longer needed), so the SAM-to-BAM sorting step may not exist in newer releases.
- No software version numbers are recorded for the tools/modules in this changelog, creating a reproducibility gap for pinning tool versions.

## Evidence

- [intro] minimap2 aligner choice: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
- [intro] samtools sort step: "- [SAMTOOLS SORT](#samtools-sort) - Sort alignment and convert sam file into bam file"
- [intro] bioperl spurious alignment filter: "- [BIOPERL](#bioperl) - Remove spurious alignments (uLTRA path only"
- [readme] TAMA purpose: [evidence span withheld — anchor status: external_doc]
- [discussion] uLTRA malformed GTF limitation: "spurious alignments produced by uLTRA if a malformed GTF is used"
- [discussion] samtools sort removal: "Therefore, `samtools sort` module is not needed anymore and has been removed."
