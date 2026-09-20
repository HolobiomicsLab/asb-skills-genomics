---
name: partial-order-alignment-consensus
description: Use when error-prone ONT or PacBio long reads must be converted into
  accurate per-haplotype consensus sequences for candidate repetitive regions (e.g.,
  BED-defined [S,E] intervals) before motif profiling.
when_to_use_negative:
- The goal is de novo discovery of repetitive regions from raw reads — use the upstream
  SENSoR Shannon-entropy stage (20 bp windows, entropy threshold 1.23) instead of
  consensus generation
- Repeats lie in unassembled regions of the genome (e.g., human centromeres and telomeres),
  which TRiCoLOR cannot identify
- Reads do not entirely span the region of interest; partially spanning reads are
  not fetched for POA consensus
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0157
tools:
- name: TRiCoLOR REFER
  role: Module that fetches haplotype-specific reads entirely spanning each BED region,
    trims them to [S, E], and orchestrates consensus generation
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: SPOA
  role: SIMD-accelerated partial order alignment framework executed on trimmed haplotype-specific
    read sets to compute per-region consensus sequences
- name: TRiCoLOR
  role: Parent tandem repeat profiling pipeline whose REFER stage performs POA-based
    consensus generation from haplotype-resolved long-read alignments
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:b3db6103e3dfe9b0255c3cb23cb47ad04b17a23123e75e5ca90aa5376a0ddfae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# partial-order-alignment-consensus

## Summary

Generates haplotype-specific low-error consensus sequences for candidate repetitive regions by trimming haplotype-resolved long reads to a region of interest and running SIMD-accelerated partial order alignment (SPOA), as used in the TRiCoLOR REFER stage.

## When to use

Use when error-prone ONT or PacBio long reads must be converted into accurate per-haplotype consensus sequences for candidate repetitive regions (e.g., BED-defined [S,E] intervals) before motif profiling.

## When NOT to use

- The goal is de novo discovery of repetitive regions from raw reads — use the upstream SENSoR Shannon-entropy stage (20 bp windows, entropy threshold 1.23) instead of consensus generation
- Repeats lie in unassembled regions of the genome (e.g., human centromeres and telomeres), which TRiCoLOR cannot identify
- Reads do not entirely span the region of interest; partially spanning reads are not fetched for POA consensus

## Inputs

- Candidate repetitive regions in BED format (regions R = [S, E])
- Haplotype-resolved BAM files of error-prone long-read alignments (ONT or PacBio, e.g., minimap2-aligned to GRCh38)
- Sequenced long reads spanning the candidate regions

## Outputs

- consensus_sequences.fasta containing one haplotype-specific low-error consensus sequence per candidate region per haplotype
- Per-haplotype consensus sequences with ~77% (ONT) and ~88% (PB) error reduction relative to raw reads

## How to apply

For each candidate region R = [S, E] in the input BED file, fetch from the haplotype-specific BAM files all sequencing reads that entirely span the region, then trim each read so only the sequence between coordinates S and E is retained — trimming reduces runtime of the downstream POA step. Run SPOA, the SIMD-accelerated partial order alignment framework, on the trimmed haplotype-specific read sets to compute a highly accurate consensus per region per haplotype. Rationale: raw long reads carry high error rates (~15% ONT), and POA consensus achieves an approximate error reduction of ~77% for ONT and ~88% for PacBio reads, making the subsequent RegEx-based motif/multiplicity resolution reliable. Write the consensus sequences to consensus_sequences.fasta, one per candidate region per haplotype.

## Related tools

- **TRiCoLOR REFER** (Module that fetches haplotype-specific reads entirely spanning each BED region, trims them to [S, E], and orchestrates consensus generation) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **SPOA** (SIMD-accelerated partial order alignment framework executed on trimmed haplotype-specific read sets to compute per-region consensus sequences)
- **TRiCoLOR** (Parent tandem repeat profiling pipeline whose REFER stage performs POA-based consensus generation from haplotype-resolved long-read alignments) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Consensus sequences achieve error reduction consistent with the reported ~77% (ONT) and ~88% (PB) improvement over raw error-prone long reads
- consensus_sequences.fasta contains exactly one consensus entry per candidate region per haplotype
- Each consensus derives only from reads that entirely span the region R = [S, E], trimmed to the S–E interval
- Downstream RegEx-based motif resolution succeeds on the consensus, resolving repeat motif length (≤6 bp) and multiplicity consistent with period size expectations

## Limitations

- Consensus accuracy depends on sufficient read depth: in the article, low-entropy regions were filtered for mean coverage >8 (ONT) or >10× (PB) before downstream analysis
- Only reads entirely spanning the region are used; partial-span reads are discarded, so long expanded repeats may lack spanning reads
- TRiCoLOR has no support for polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected upstream, so no consensus can be generated there

## Evidence

- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] SPOA error reduction of ~77% and ~88% for ONT and PB: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] REFER module profiles TRs in haplotype-resolved BAM files: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [methods] TRiCoLOR requires haplotype-resolved long-read alignments as input: "TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input"
- [abstract] TRiCoLOR identifies repetitive regions without prior knowledge of motifs or locations: "The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner."
