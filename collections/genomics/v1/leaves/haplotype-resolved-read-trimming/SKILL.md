---
name: haplotype-resolved-read-trimming
description: Use when you have haplotype-resolved BAM files and a BED of candidate
  repetitive regions R = [S, E], and need to prepare span-only read subsets for per-region,
  per-haplotype consensus generation (e.g., TRiCoLOR REFER).
when_to_use_negative:
- Regions not spanned end-to-end by any long read in a given haplotype — no entirely
  spanning read can be fetched, so no trimmed read set or consensus can be produced
- Genomic regions that are unassembled (e.g., human centromeres and telomeres) — TRiCoLOR
  cannot identify repeats there
- Haploid chromosomes (e.g., human Y chromosome) or polyploid species — outside TRiCoLOR's
  current diploid haplotype-resolved design
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TRiCoLOR REFER
  role: Module that fetches reads entirely spanning each BED region and trims them
    to coordinates S–E prior to POA
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: SPOA
  role: SIMD-accelerated partial order alignment framework run on the trimmed haplotype-specific
    read sets to compute per-region consensus sequences
- name: TRiCoLOR
  role: Pipeline wrapper providing the haplotype-resolved BAM inputs and orchestrating
    the REFER stage
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:8527056399b26badfdda9f8c945785648053e9f8241a386225c6474cd0ea6e0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# haplotype-resolved-read-trimming

## Summary

Trims haplotype-specific long reads (ONT or PacBio BAM alignments) so that only the sequence between the coordinates S and E of a candidate repetitive region is retained, before partial order alignment (POA) computes per-haplotype consensus sequences. Trimming reduces runtime of the downstream SPOA step without sacrificing consensus accuracy.

## When to use

Use when you have haplotype-resolved BAM files and a BED of candidate repetitive regions R = [S, E], and need to prepare span-only read subsets for per-region, per-haplotype consensus generation (e.g., TRiCoLOR REFER).

## When NOT to use

- Regions not spanned end-to-end by any long read in a given haplotype — no entirely spanning read can be fetched, so no trimmed read set or consensus can be produced
- Genomic regions that are unassembled (e.g., human centromeres and telomeres) — TRiCoLOR cannot identify repeats there
- Haploid chromosomes (e.g., human Y chromosome) or polyploid species — outside TRiCoLOR's current diploid haplotype-resolved design

## Inputs

- Haplotype-resolved long-read alignment BAM files (ONT or PacBio, e.g., minimap2-aligned to GRCh38)
- BED file of candidate repetitive regions R = [S, E] (e.g., low-entropy regions from the TRiCoLOR SENSoR module)

## Outputs

- Trimmed per-region, per-haplotype read sets (sequences between S and E only)
- consensus_sequences.fasta with one SPOA-derived consensus sequence per candidate region per haplotype

## How to apply

For each region R = [S, E] in the input BED file, fetch from the haplotype-specific BAM files all sequencing reads that entirely span the region — partial-spanning reads are excluded so the trimmed sequence fully covers S to E. Trim each fetched read so that only the subsequence between coordinates S and E is retained; this is done per haplotype, so downstream consensus sequences remain haplotype-specific. The rationale is computational: trimming reduces the runtime of the downstream SPOA partial order alignment step while preserving the bases needed for consensus. Run SPOA on the trimmed haplotype-specific read sets to compute a highly accurate consensus per region per haplotype, with an approximate error reduction of ~77% for ONT and ~88% for PB reads. Write results to consensus_sequences.fasta, one consensus per candidate region per haplotype.

## Related tools

- **TRiCoLOR REFER** (Module that fetches reads entirely spanning each BED region and trims them to coordinates S–E prior to POA) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **SPOA** (SIMD-accelerated partial order alignment framework run on the trimmed haplotype-specific read sets to compute per-region consensus sequences)
- **TRiCoLOR** (Pipeline wrapper providing the haplotype-resolved BAM inputs and orchestrating the REFER stage) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Every trimmed read in a region's read set spans the full interval [S, E] after trimming (no partial coverage of the target coordinates)
- consensus_sequences.fasta contains exactly one consensus sequence per candidate region per haplotype
- Consensus error rate is reduced by approximately 77% for ONT reads and 88% for PB reads relative to raw error-prone long reads
- Downstream SPOA runtime on trimmed read sets is reduced compared to running POA on the full untrimmed reads
- Consensus sequences are haplotype-specific: per-haplotype BAMs yield distinct consensus calls consistent with allele-specific repeat content

## Limitations

- Requires reads that entirely span the candidate region; reads covering only part of [S, E] cannot be trimmed and used
- TRiCoLOR does not support polyploid species or haploid chromosomes such as the human Y chromosome
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected or reconstructed
- Consensus accuracy depends on long-read error profiles; the reported ~77%/~88% error reduction is technology-specific (ONT vs PB)

## Evidence

- [full_text] fetch from the haplotype-specific BAM files all sequencing reads that entirely span the region: [evidence span withheld — anchor status: not_found]
- [full_text] Trim each fetched read so that only the sequence between coordinates S and E is retained: [evidence span withheld — anchor status: not_found]
- [methods] consensus with an approximate error reduction of ~77% and ~88% for ONT and PB: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
