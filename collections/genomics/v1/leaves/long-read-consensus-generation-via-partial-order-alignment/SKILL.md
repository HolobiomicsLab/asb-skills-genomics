---
name: long-read-consensus-generation-via-partial-order-alignment
description: Use when error-prone haplotype-resolved long-read alignments (BAM) over
  candidate repetitive regions must be condensed into accurate per-haplotype consensus
  sequences before motif/multiplicity profiling or downstream comparison to GRCh38.
when_to_use_negative:
- Input is raw unassembled FASTQ without prior haplotype-resolved alignment to a reference
  — TRiCoLOR requires haplotype-resolved BAM input
- Target regions are in unassembled parts of the genome (centromeres, telomeres),
  where TRiCoLOR cannot identify repeats
- Data are polyploid or from haploid chromosomes (e.g., human Y), which TRiCoLOR does
  not support
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: Pipeline that invokes SPOA internally to compute haplotype-specific consensus
    sequences from haplotype-resolved BAM input
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Upstream module producing the low-entropy candidate repetitive regions whose
    read piles are condensed into consensus sequences
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns the generated haplotype consensus sequences to GRCh38 for downstream
    profiling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:d9aa172763e5dc24f96769b3a8a4f969f40080c0d0ae61846a81927cb2bbbe4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Long-read consensus generation via partial order alignment

## Summary

Compute haplotype-specific, low-error consensus sequences from error-prone ONT or PacBio long reads aligned to repetitive loci, using partial order alignment (SPOA) as implemented inside the TRiCoLOR pipeline, reducing raw read error by ~77% (ONT) and ~88% (PB) before motif resolution.

## When to use

Use when error-prone haplotype-resolved long-read alignments (BAM) over candidate repetitive regions must be condensed into accurate per-haplotype consensus sequences before motif/multiplicity profiling or downstream comparison to GRCh38.

## When NOT to use

- Input is raw unassembled FASTQ without prior haplotype-resolved alignment to a reference — TRiCoLOR requires haplotype-resolved BAM input
- Target regions are in unassembled parts of the genome (centromeres, telomeres), where TRiCoLOR cannot identify repeats
- Data are polyploid or from haploid chromosomes (e.g., human Y), which TRiCoLOR does not support

## Inputs

- Haplotype-resolved long-read alignment BAM (ONT or PacBio) restricted to candidate repetitive regions from the SENSoR BED scan
- GRCh38 reference genome for post-consensus alignment with minimap2

## Outputs

- Haplotype-specific low-error consensus sequences per repetitive region
- minimap2 alignments of consensus sequences to GRCh38
- Haplotype-specific BAM files with aligned consensus sequences consumed by the RegEx motif/multiplicity matcher

## How to apply

Start from haplotype-resolved long-read alignments (BAM) of reads spanning candidate repetitive regions previously flagged by the SENSoR entropy scan (20 bp windows, entropy <= 1.23, >= 5 supporting reads, merge within 100 bp). Group the reads by haplotype and run partial order alignment (POA, via SPOA) on each read pile to derive one consensus sequence per haplotype; SPOA's iterative alignment-to-partial-order graph suppresses random sequencing noise, yielding an approximate error reduction of ~77% for ONT and ~88% for PacBio data. Align the resulting consensus sequences to the GRCh38 reference with minimap2, then hand them to the RegEx-based approximate string matcher (edit distance <= 1) to resolve repeat motif and multiplicity, resolving overlaps with the N-gram model. Rationale: the consensus step is the noise-reduction hinge of TRiCoLOR — without it, the <= 1 edit-distance motif matcher cannot distinguish true imperfect repeats from raw-read errors.

## Related tools

- **TRiCoLOR** (Pipeline that invokes SPOA internally to compute haplotype-specific consensus sequences from haplotype-resolved BAM input) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (Upstream module producing the low-entropy candidate repetitive regions whose read piles are condensed into consensus sequences) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns the generated haplotype consensus sequences to GRCh38 for downstream profiling)

## Evaluation signals

- Consensus generation succeeds only if every SENSoR-predicted region yields at least one consensus sequence per haplotype (diploid: two per region)
- Approximate error reduction should be in the expected range of ~77% for ONT and ~88% for PB input
- Downstream RegEx matcher must resolve a repeat motif and multiplicity for each consensus; empty motif output across all regions signals consensus-quality failure
- Consensus-to-GRCh38 minimap2 alignments must map back to the original genomic coordinates of the candidate region
- Final call set should contain only TRs >= 50 bp varying between haplotypes or the reference, stored in BCF-compliant format with accompanying BED files

## Limitations

- Consensus accuracy depends on sufficient read coverage; low-entropy regions were filtered to mean coverage >8x (ONT) and >10x (PB) before profiling
- Cannot produce consensus for repeats in unassembled regions of the genome (e.g., centromeres and telomeres)
- No support for polyploid species or haploid chromosomes (e.g., human Y chromosome)

## Evidence

- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] SPOA error reduction on ONT and PB reads: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] Input requirement for consensus generation: "TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input"
- [intro] Downstream consumption of consensus sequences: "processed by means of a fast regular expression (RegEx)-based approximate string-matching algorithm to resolve repeat motif and multiplicity of the discovered TRs"
- [readme] Installation route for running the SPOA-bearing pipeline: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
