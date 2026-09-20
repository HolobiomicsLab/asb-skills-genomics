---
name: shannon-entropy-sequence-scanning
description: Use when starting from haplotype-resolved long-read BAM alignments and
  needing de novo discovery of candidate repetitive/tandem-repeat regions without
  any prior knowledge of motifs or repeat locations.
when_to_use_negative:
- Input lacks haplotype-resolved long-read alignments — TRiCoLOR requires haplotype-resolved
  BAM files as input, so short-read or non-phased BAMs are out of scope.
- Goal is to detect repeats in unassembled regions such as human centromeres and telomeres,
  which TRiCoLOR cannot identify.
- Input is polyploid species or haploid chromosomes (e.g., human Y chromosome), which
  TRiCoLOR does not support.
- You already have an annotated tandem-repeat catalog or known motifs — REFER profiling
  of specific loci may be more appropriate than de novo entropy scanning.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR SENSoR
  role: Core module that computes Shannon entropy over non-overlapping 20 bp sliding
    windows per read to identify low-entropy candidate repetitive segments
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: Parent pipeline whose downstream REFER module consumes the SENSoR BED output
    to profile tandem repeats
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: pysam
  role: Library used to load and iterate over haplotype-resolved BAM alignments read-by-read
    during entropy scanning
- name: minimap2
  role: Aligner used upstream to produce the ONT long-read BAM alignments that SENSoR
    scans
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:c4379dd801995bc3599c0f728863ee4b19286d51dedceb143ab54ea62c0cfc6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR SENSoR
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# shannon-entropy-sequence-scanning

## Summary

Reconstructs the SENSoR module of TRiCoLOR, which scans haplotype-resolved ONT long-read alignments with a Shannon-entropy test on 20 bp non-overlapping windows to identify candidate repetitive regions de novo, emitting a sorted 3-column BED file.

## When to use

Use when starting from haplotype-resolved long-read BAM alignments and needing de novo discovery of candidate repetitive/tandem-repeat regions without any prior knowledge of motifs or repeat locations.

## When NOT to use

- Input lacks haplotype-resolved long-read alignments — TRiCoLOR requires haplotype-resolved BAM files as input, so short-read or non-phased BAMs are out of scope.
- Goal is to detect repeats in unassembled regions such as human centromeres and telomeres, which TRiCoLOR cannot identify.
- Input is polyploid species or haploid chromosomes (e.g., human Y chromosome), which TRiCoLOR does not support.
- You already have an annotated tandem-repeat catalog or known motifs — REFER profiling of specific loci may be more appropriate than de novo entropy scanning.

## Inputs

- Haplotype-resolved long-read BAM alignment files (e.g., ONT reads aligned to GRCh38 with minimap2)
- Parameters: window size 20 bp (default), entropy threshold 1.23 (default), minimum read support 5, merge distance 100 bp

## Outputs

- candidate_repetitive_regions.bed — sorted, 3-column BED file (chromosome, start, end) of merged low-entropy candidate repetitive regions (~160,000-~260,000 regions per sample, mean length ~900 bp before coverage filtering)

## How to apply

Load the haplotype-resolved BAM files with pysam and iterate over each sequencing read's aligned genomic coordinates, computing the Shannon entropy of each non-overlapping sliding window of 20 bp (the pre-trained default window size) along the aligned sequence. Retain windows with entropy <= 1.23, the default threshold that 'efficiently discriminates between repetitive and non-repetitive DNA sequences', keeping only windows supported by >= 5 independent reads. Merge retained window coordinates lying within 100 bp of one another into contiguous candidate repetitive regions, then write the merged regions to a sorted, 3-column BED file (chromosome, start, end) named candidate_repetitive_regions.bed. On the paper's real ONT data this produced ~160,000-~260,000 candidate regions per sample with a mean region length of ~900 bp, which downstream REFER profiling (consensus via POA, RegEx-based motif resolution) consumes as input.

## Related tools

- **TRiCoLOR SENSoR** (Core module that computes Shannon entropy over non-overlapping 20 bp sliding windows per read to identify low-entropy candidate repetitive segments) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Parent pipeline whose downstream REFER module consumes the SENSoR BED output to profile tandem repeats) — https://github.com/davidebolo1993/TRiCoLOR
- **pysam** (Library used to load and iterate over haplotype-resolved BAM alignments read-by-read during entropy scanning)
- **minimap2** (Aligner used upstream to produce the ONT long-read BAM alignments that SENSoR scans)

## Evaluation signals

- Output is a valid sorted 3-column BED file whose intervals are non-overlapping.
- Every constituent window satisfied entropy <= 1.23 and >= 5 independent reads of support before merging.
- Number of candidate regions per sample falls in the reported ~160,000-~260,000 range (for mean coverage >8 ONT data), with mean region length ~900 bp.
- Merged regions correspond to coordinates retained from windows within 100 bp of one another.
- Downstream REFER module can consume the BED and profile motifs (length <= 6, >= 5 perfect repeats) at the flagged coordinates.

## Limitations

- The entropy threshold and window size were empirically estimated; parameter values for profiling mini-satellites have not been validated — the tool has been extensively applied only to micro-satellites.
- Repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) cannot be identified.
- No support for polyploid species or haploid chromosomes such as the human Y chromosome.
- Low-entropy region counts depend on coverage — ONT regions were reduced from ~260,000 to ~150,000 after filtering for mean coverage >8 (and >10x for PB data).

## Evidence

- [methods] SENSoR module uses Shannon entropy to detect low-entropy regions: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [methods] 20 bp sliding windows and multi-read coordinate retention: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default). Genomic coordinates of windows in which multiple reads"
- [methods] Entropy threshold 1.23 discriminates repetitive from non-repetitive DNA: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [intro] Low-entropy regions identified in sequenced long reads (intro framing): "Once low-entropy repetitive regions have been identified in sequenced long reads"
- [results] Coverage filtering reduces low-entropy region count: "after filtering for regions with mean coverage >8"
- [readme] TRiCoLOR availability and repository location: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
