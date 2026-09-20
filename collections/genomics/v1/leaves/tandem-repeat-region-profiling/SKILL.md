---
name: tandem-repeat-region-profiling
description: Use when you need to identify and genotype tandem repeats de novo from
  haplotype-resolved BAM files of error-prone long reads (ONT or PacBio), without
  prior knowledge of repeat motifs or genomic locations.
when_to_use_negative:
- You need to detect repeats in unassembled genomic regions such as human centromeres
  and telomeres — TRiCoLOR cannot identify repeats there.
- Your data is from a polyploid species or a haploid chromosome (e.g., the human Y
  chromosome) — TRiCoLOR assumes diploid haplotype-resolved input.
- Your input is short-read (Illumina) alignments or non-haplotype-resolved BAMs —
  the workflow requires haplotype-resolved error-prone long-read alignments.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0157
tools:
- name: TRiCoLOR
  role: 'End-to-end pipeline: SENSoR entropy scanning, REFER POA consensus generation,
    and RegEx-based TR motif/multiplicity profiling'
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Identifies candidate low-entropy repetitive regions from haplotype-resolved
    alignments and emits the candidate BED
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: Fetches and trims haplotype-spanning reads per region and runs POA to produce
    haplotype-specific consensus sequences
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: SIMD-accelerated partial order alignment framework used to compute the consensus
    sequence per region and haplotype
- name: minimap2
  role: Recommended aligner for producing the ONT/PacBio long-read alignments consumed
    by TRiCoLOR
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:9f5ce687a3a54d6446cd135ca86e682bcb9c5d1506f2416200c83d1459c17039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-repeat-region-profiling

## Summary

De novo profiling of tandem repeats from haplotype-resolved ONT/PacBio long-read alignments using TRiCoLOR: Shannon-entropy scanning identifies candidate repetitive regions, POA-based consensus sequences correct raw read errors, and a RegEx-based approximate string-matching step resolves repeat motif and multiplicity per haplotype.

## When to use

Use when you need to identify and genotype tandem repeats de novo from haplotype-resolved BAM files of error-prone long reads (ONT or PacBio), without prior knowledge of repeat motifs or genomic locations.

## When NOT to use

- You need to detect repeats in unassembled genomic regions such as human centromeres and telomeres — TRiCoLOR cannot identify repeats there.
- Your data is from a polyploid species or a haploid chromosome (e.g., the human Y chromosome) — TRiCoLOR assumes diploid haplotype-resolved input.
- Your input is short-read (Illumina) alignments or non-haplotype-resolved BAMs — the workflow requires haplotype-resolved error-prone long-read alignments.

## Inputs

- Haplotype-resolved long-read alignment BAM files (ONT or PacBio reads aligned e.g. with minimap2 to GRCh38)
- Reference genome FASTA (e.g., GRCh38)

## Outputs

- candidate_repetitive_regions.bed — sorted 3-column BED (chromosome, start, end) of merged low-entropy regions
- consensus_sequences.fasta — haplotype-specific low-error consensus sequence per candidate region per haplotype
- BCF-compliant file of genotyped tandem repeats (length >= 50 bp) varying between haplotypes or the reference

## How to apply

First run the SENSoR module: load haplotype-resolved BAMs with pysam, compute Shannon entropy in non-overlapping 20 bp sliding windows along each read's aligned sequence, retain windows with entropy <= 1.23 supported by >= 5 reads, merge windows within 100 bp, and write a sorted 3-column BED of candidate repetitive regions. Then run the REFER module: for each region [S, E], fetch all haplotype-specific reads fully spanning the region, trim them to S-E, and run SPOA to compute low-error haplotype-specific consensus sequences (approximately 77% and 88% error reduction for ONT and PB reads, respectively). Finally, apply the RegEx-based approximate string-matching profiler: identify perfectly repeated motifs of length <= 6 repeated >= 5 times, then search approximate motif repetitions up to an edit distance <= 1, resolve competing overlapping predictions with an N-gram model favoring the most frequent perfect repeat motif, keep TRs >= 50 bp that vary between haplotypes or the reference, and store them in BCF format.

## Related tools

- **TRiCoLOR** (End-to-end pipeline: SENSoR entropy scanning, REFER POA consensus generation, and RegEx-based TR motif/multiplicity profiling) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (Identifies candidate low-entropy repetitive regions from haplotype-resolved alignments and emits the candidate BED) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (Fetches and trims haplotype-spanning reads per region and runs POA to produce haplotype-specific consensus sequences) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (SIMD-accelerated partial order alignment framework used to compute the consensus sequence per region and haplotype)
- **minimap2** (Recommended aligner for producing the ONT/PacBio long-read alignments consumed by TRiCoLOR)

## Evaluation signals

- Output BED is a valid, sorted, non-overlapping 3-column BED file whose constituent windows each satisfied entropy <= 1.23 and >= 5-read support before 100 bp merging.
- Consensus sequences achieve error reduction consistent with the reported ~77% (ONT) and ~88% (PB) improvement over raw error-prone long reads.
- On real data, expect on the order of ~160,000–~260,000 candidate repetitive regions per sample with mean region length of ~900 bp before coverage filtering.
- All reported TR calls are >= 50 bp and BCF-compliant, with competing overlapping predictions resolved by the N-gram model favoring the most frequent perfect repeat motif.
- Genotypes are haplotype-specific and can be checked for Mendelian consistency in trio sequencing studies.

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres).
- No support for polyploid species or haploid chromosomes such as the human Y chromosome.
- Parameters for profiling mini-satellites are not validated; the tool has been extensively applied so far only to micro-satellites.
- Candidate regions must be covered by reads entirely spanning the region; low-coverage regions are filtered out (mean coverage > 8 for ONT, > 10x for PB in the reported analyses).

## Evidence

- [methods] TRiCoLOR SENSoR: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [methods] entropy threshold: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] SPOA error reduction: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] motif profiling: "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [intro] consensus purpose: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [abstract] de novo capability: "The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner."
- [readme] installability: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
