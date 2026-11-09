---
name: long-read-splice-aware-alignment
description: Use when FLNC reads (post CCS/LIMA/refine and TAMA polyA cleanup, in
  FASTA form) must be spliced-aligned to a genome and a reference GTF may or may not
  exist, requiring a dispatch between uLTRA and minimap2.
when_to_use_negative:
- Input is raw PacBio subreads — CCS, LIMA, isoseq3 refine, and TAMA polyA cleanup
  must run first to produce FLNC FASTA.
- The target is transcriptome (not genome) alignment — both uLTRA and minimap2 here
  are configured for splice-aware genome mapping.
- A malformed GTF is supplied for the uLTRA path without running the BioPerl cleanup
  — spurious alignments will propagate downstream.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: uLTRA
  role: splice-aware aligner guided by exon annotation database; preferred when a
    GTF is available because it is particularly accurate for small exons
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: splice-aware fallback aligner used when no genome annotation is available
  repo: https://github.com/lh3/minimap2
- name: BioPerl
  role: removes spurious uLTRA alignments whose CIGAR strings contain N gaps (uLTRA
    path only)
  repo: https://bioperl.org/
- name: nf-core/isoseq
  role: pipeline implementing the conditional uLTRA/minimap2 dispatch, chunking, and
    output directory layout
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:51133dffc946ffd30d1519e51104ac2d9a44a167e9a951352a71ec27afd829cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/ultra
  tools:
  - BAMTools
  - BioPerl
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - TAMA
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# long-read-splice-aware-alignment

## Summary

Maps PacBio Iso-Seq FLNC reads onto a reference genome with a splice-aware aligner — uLTRA when a GTF annotation is available (better small-exon detection) or minimap2 otherwise — as implemented in the nf-core/isoseq pipeline.

## When to use

Use when FLNC reads (post CCS/LIMA/refine and TAMA polyA cleanup, in FASTA form) must be spliced-aligned to a genome and a reference GTF may or may not exist, requiring a dispatch between uLTRA and minimap2.

## When NOT to use

- Input is raw PacBio subreads — CCS, LIMA, isoseq3 refine, and TAMA polyA cleanup must run first to produce FLNC FASTA.
- The target is transcriptome (not genome) alignment — both uLTRA and minimap2 here are configured for splice-aware genome mapping.
- A malformed GTF is supplied for the uLTRA path without running the BioPerl cleanup — spurious alignments will propagate downstream.

## Inputs

- polyA-trimmed FLNC reads in FASTA format (TAMA GSTAMA_POLYACLEANUP output, gunzipped for the uLTRA path)
- reference genome (FASTA)
- optional reference genome annotation (GTF)

## Outputs

- <sample>.chunk<X>.sam per-chunk splice-aware alignment files under 06.2_ULTRA/ (uLTRA) or 06_MINIMAP2/ (minimap2)
- sorted BAM alignments after samtools sort (input to TAMA merge)

## How to apply

Load the polyA-cleaned FLNC reads (decompress with gunzip for the uLTRA path), the reference genome, and the optional reference GTF. Apply the dispatch condition: if a GTF is present, run uLTRA in genome+annotation mode since it uses the annotation database to accurately align long reads to small exons; otherwise fall back to minimap2 in splice-aware mode. Each tool chunks reads into per-sample per-chunk alignments, written as <sample>.chunk<X>.sam files under branch-specific directories (06.2_ULTRA/ or 06_MINIMAP2/). For the uLTRA path, follow with BioPerl to remove spurious alignments (uLTRA can emit CIGAR strings with N gaps when a malformed GTF is used), then sort alignments and convert SAM to BAM with samtools sort. Validate by checking that every expected <sample>.chunk<X>.sam file exists and is non-empty in the branch output directory.

## Related tools

- **uLTRA** (splice-aware aligner guided by exon annotation database; preferred when a GTF is available because it is particularly accurate for small exons) — https://github.com/ksahlin/ultra
- **minimap2** (splice-aware fallback aligner used when no genome annotation is available) — https://github.com/lh3/minimap2
- **BioPerl** (removes spurious uLTRA alignments whose CIGAR strings contain N gaps (uLTRA path only)) — https://bioperl.org/
- **nf-core/isoseq** (pipeline implementing the conditional uLTRA/minimap2 dispatch, chunking, and output directory layout) — https://github.com/nf-core/isoseq

## Examples

```
nextflow run nf-core/isoseq -profile test --aligner ultra
```

## Evaluation signals

- Every expected <sample>.chunk<X>.sam file exists and is non-empty in the branch output directory (06.2_ULTRA/ or 06_MINIMAP2/).
- The dispatch condition honored the annotation availability: uLTRA ran only when a GTF was provided, minimap2 only when it was not.
- Sorted BAM output from samtools sort is produced and is consumable by the downstream TAMA (GSTAMA merge) step.
- For the uLTRA path, the BioPerl cleanup removed alignments with N-carrying CIGAR strings before sorting.
- The full pipeline run completes and the downstream 10_GSTAMA_MERGE/<sample>.bed merged annotation is produced in valid BED12 format.

## Limitations

- uLTRA produces spurious alignments if a malformed GTF is used; removing the BioPerl module would stop the pipeline in that case.
- uLTRA requires the FLNC FASTA to be decompressed (gunzip step, uLTRA path only).
- Newer pipeline releases removed the uLTRA pipeline process, the bioperl module, and the samtools sort module, so this skill's branch layout may not hold in later versions.
- No software version numbers are recorded for the tools/modules, which limits reproducible pinning of aligner versions.

## Evidence

- [intro] uLTRA preference with annotation: [evidence span withheld — anchor status: external_doc]
- [readme] uLTRA small exon accuracy: [evidence span withheld — anchor status: external_doc]
- [intro] splice-aware genome mapping: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
- [intro] BioPerl spurious alignment cleanup: "- [BIOPERL](#bioperl) - Remove spurious alignments (uLTRA path only"
- [discussion] uLTRA malformed GTF failure mode: "spurious alignments produced by uLTRA if a malformed GTF is used"
