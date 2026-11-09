---
name: bam-alignment-read-parsing
description: Use when input is haplotype-resolved ONT/PacBio BAM alignments and the
  goal is de novo detection of candidate repetitive regions without known motifs or
  prior TR annotations.
when_to_use_negative:
- Regions of interest are already annotated as tandem repeats and motif-aware profiling
  (e.g. REFER module) can be applied directly to the alignments
- Repeats lie in unassembled genomic regions such as human centromeres and telomeres,
  which TRiCoLOR cannot identify
- Input alignments are not haplotype-resolved or come from polyploid/haploid chromosomes
  (e.g. the human Y chromosome), which are outside TRiCoLOR's scope
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR SENSoR
  role: SENSoR module that uses Shannon entropy of DNA sequences to identify candidate
    repetitive segments during the BAM read-scan
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: overall pipeline whose first stage consumes haplotype-resolved long-read alignments
    and produces the candidate repetitive-regions BED
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: pysam
  role: Python library used to load the haplotype-resolved BAM files and iterate over
    each sequencing read's aligned genomic coordinates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:92f4d688a46123de8624d5554c167fecf0a423e7d7a34ecd111401a36badb4b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR SENSoR
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bam-alignment-read-parsing

## Summary

Iterate over haplotype-resolved long-read alignments in BAM format (via pysam) to compute per-read Shannon entropy over non-overlapping 20 bp windows, flagging low-entropy windows as candidate repetitive sequence for downstream tandem-repeat discovery (TRiCoLOR SENSoR step).

## When to use

Use when input is haplotype-resolved ONT/PacBio BAM alignments and the goal is de novo detection of candidate repetitive regions without known motifs or prior TR annotations.

## When NOT to use

- Regions of interest are already annotated as tandem repeats and motif-aware profiling (e.g. REFER module) can be applied directly to the alignments
- Repeats lie in unassembled genomic regions such as human centromeres and telomeres, which TRiCoLOR cannot identify
- Input alignments are not haplotype-resolved or come from polyploid/haploid chromosomes (e.g. the human Y chromosome), which are outside TRiCoLOR's scope

## Inputs

- haplotype-resolved long-read alignment BAM file(s) (ONT or PacBio, e.g. minimap2-aligned to GRCh38)
- optional parameter settings: window size (20 bp default), entropy threshold (1.23 default), read-support cutoff (5 reads), merge distance (100 bp)

## Outputs

- sorted 3-column BED file (chromosome, start, end) of merged candidate repetitive regions, named candidate_repetitive_regions.bed
- typically ~160,000-~260,000 candidate repetitive regions per sample with mean region length of ~900 bp

## How to apply

Load the haplotype-resolved BAM files with pysam and iterate over each read's aligned genomic coordinates. For every read, compute the Shannon entropy of each non-overlapping sliding window of 20 bp (the paper's pre-trained default window size) along the aligned sequence. Retain the genomic coordinates of windows with entropy <= 1.23 (the default threshold shown to efficiently discriminate repetitive from non-repetitive DNA on synthetic ONT and PB reads), keeping only windows supported by >= 5 independent reads. Merge retained window coordinates lying within 100 bp of one another into contiguous candidate repetitive regions and emit them as a sorted, 3-column BED file named candidate_repetitive_regions.bed (chromosome, start, end). The rationale is that repetitive DNA collapses sequence complexity, so a 1.23-bit entropy floor over 20 bp windows marks repetitive stretches while multi-read support suppresses spurious low-entropy calls from sequencing error.

## Related tools

- **TRiCoLOR SENSoR** (SENSoR module that uses Shannon entropy of DNA sequences to identify candidate repetitive segments during the BAM read-scan) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (overall pipeline whose first stage consumes haplotype-resolved long-read alignments and produces the candidate repetitive-regions BED) — https://github.com/davidebolo1993/TRiCoLOR
- **pysam** (Python library used to load the haplotype-resolved BAM files and iterate over each sequencing read's aligned genomic coordinates)

## Examples

```
import pysam; bam = pysam.AlignmentFile("sample.haplotagged.bam", "rb"); [compute entropy per 20bp window for read.query_alignment_sequence for read in bam.fetch()]
```

## Evaluation signals

- Output is a valid 3-column BED file named candidate_repetitive_regions.bed with sorted, non-overlapping intervals
- Every merged region's constituent windows each satisfied entropy <= 1.23 and >= 5 independent read support
- Region counts and sizes fall in the reported range: ~160,000-~260,000 candidate regions per sample with mean region length ~900 bp
- After coverage filtering (mean coverage >8 for ONT, >10x for PB) region counts shrink toward ~150,000, as reported for real HGSVC samples
- Windows are computed over non-overlapping 20 bp bins along each read's aligned sequence, so merged region boundaries align to multiples consistent with the window grid

## Limitations

- Only windows in aligned portions of reads are scanned; reads must be mapped (minimap2 to GRCh38 in the paper) before entropy computation
- Low-entropy candidate regions must subsequently be filtered for mean coverage (>8 for ONT, >10x for PB) before profiling, otherwise counts are inflated by noise
- Cannot detect repeats in unassembled regions (centromeres, telomeres) and has no support for polyploid species or haploid chromosomes such as the human Y

## Evidence

- [methods] Shannon entropy windows and threshold: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default)"
- [methods] Entropy threshold discriminative power: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] SENSoR module purpose: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [methods] TRiCoLOR input requirement: "TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input"
- [readme] Installation/source availability: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
