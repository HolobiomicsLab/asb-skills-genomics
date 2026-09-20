---
name: genomic-interval-merging
description: Use when per-read, window-level low-entropy or feature-bearing coordinates
  must be coalesced into non-overlapping genomic intervals (e.g., BED regions) prior
  to tandem repeat discovery or coverage-filtered profiling.
when_to_use_negative:
- Target regions are already annotated tandem repeats or an existing BED of candidate
  loci is supplied — use TRiCoLOR's REFER profiling step directly instead of re-deriving
  candidates.
- Repeats lie in unassembled regions of the genome (e.g., centromeres and telomeres),
  which TRiCoLOR cannot identify.
- Data are not haplotype-resolved long-read alignments; short-read or unphased BAMs
  violate the module's input assumptions.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR SENSoR
  role: Computes per-window Shannon entropy over haplotype-resolved long reads and
    produces the merged candidate repetitive region BED that this merging step finalizes
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR
  role: Containerized pipeline (Docker image davidebolo1993/tricolor) whose SENSoR
    stage emits the candidate_repetitive_regions.bed output
  repo: https://hub.docker.com/r/davidebolo1993/tricolor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:570870f619c08f18341f794d4d6787d739a1e1a7acd9692c64c32a1f68be33de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR SENSoR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Genomic interval merging for candidate repeat region construction

## Summary

Merges retained low-entropy window coordinates from haplotype-resolved long-read alignments into contiguous candidate repetitive regions, as done by TRiCoLOR's SENSoR module before repeat profiling. Coalescing nearby windows collapses per-read, per-window evidence into stable genomic intervals suitable for downstream motif discovery.

## When to use

Use when per-read, window-level low-entropy or feature-bearing coordinates must be coalesced into non-overlapping genomic intervals (e.g., BED regions) prior to tandem repeat discovery or coverage-filtered profiling.

## When NOT to use

- Target regions are already annotated tandem repeats or an existing BED of candidate loci is supplied — use TRiCoLOR's REFER profiling step directly instead of re-deriving candidates.
- Repeats lie in unassembled regions of the genome (e.g., centromeres and telomeres), which TRiCoLOR cannot identify.
- Data are not haplotype-resolved long-read alignments; short-read or unphased BAMs violate the module's input assumptions.

## Inputs

- Haplotype-resolved long-read alignment BAM files (ONT or PacBio, e.g., minimap2-aligned to GRCh38)
- Per-window retained coordinates passing the entropy <= 1.23 and >= 5-read support criteria

## Outputs

- candidate_repetitive_regions.bed — sorted, 3-column BED file (chromosome, start, end) of merged, non-overlapping candidate repetitive regions
- ~160,000–~260,000 candidate regions per sample with mean region length ~900 bp

## How to apply

Load haplotype-resolved BAM alignments (e.g., ONT reads from HGSVC samples such as HG00514, HG00733, NA19240) with pysam and compute Shannon entropy over non-overlapping 20 bp sliding windows along each aligned read. Retain only windows with entropy <= 1.23 (the default threshold shown to discriminate repetitive from non-repetitive DNA) that are supported by >= 5 independent reads. Merge retained window coordinates that lie within 100 bp of one another into contiguous candidate repetitive regions, yielding ~160,000–~260,000 regions per sample with a mean region length of ~900 bp. Sort the merged intervals and write them as a 3-column BED (chromosome, start, end), then optionally filter regions by mean coverage (>8x for ONT, >10x for PacBio) before downstream REFER-based repeat profiling.

## Related tools

- **TRiCoLOR SENSoR** (Computes per-window Shannon entropy over haplotype-resolved long reads and produces the merged candidate repetitive region BED that this merging step finalizes) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR** (Containerized pipeline (Docker image davidebolo1993/tricolor) whose SENSoR stage emits the candidate_repetitive_regions.bed output) — https://hub.docker.com/r/davidebolo1993/tricolor

## Examples

```
docker run -v $PWD:/data davidebolo1993/tricolor TRiCoLOR.py -b sample.bam -r GRCh38.fa -o /data/out
```

## Evaluation signals

- Output BED has exactly 3 columns (chromosome, start, end), is sorted, and intervals are non-overlapping after merging.
- Every merged region decomposes into constituent windows each satisfying entropy <= 1.23 and >= 5 independent reads of support.
- Merged regions per sample fall in the ~160,000–~260,000 range with mean region length ~900 bp (ONT HGSVC samples).
- After coverage filtering, ONT region counts reduce for regions with mean coverage >8 (and PB with mean coverage >10x), consistent with paper-reported numbers.
- Threshold sensitivity check: the 1.23 default entropy cutoff discriminates repetitive from non-repetitive synthetic ONT/PB sequence as reported.

## Limitations

- Candidate regions derive only from windows meeting the default 20 bp window size and 1.23 entropy threshold; alternative empirical threshold/window settings were flagged as not exhaustively reported.
- Repeats in unassembled genome regions (centromeres, telomeres) are invisible to the input alignments and thus to the merged regions.
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome).
- Regions must additionally be filtered for mean coverage (>8x ONT, >10x PB) to match the paper's reported candidate counts; unfiltered counts are inflated.

## Evidence

- [methods] Computes per-read Shannon entropy in fixed windows before coordinate retention and merging: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default). Genomic coordinates of windows in which multiple reads"
- [methods] Default entropy threshold discriminates repetitive from non-repetitive DNA: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] SENSoR module identifies candidate repetitive segments via Shannon entropy: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [results] Coverage-based filtering of low-entropy regions on real ONT data: "after filtering for regions with mean coverage >8"
- [readme] Docker availability for reproducing the SENSoR merging workflow: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
