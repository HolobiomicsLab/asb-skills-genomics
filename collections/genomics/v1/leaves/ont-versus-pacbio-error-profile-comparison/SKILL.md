---
name: ont-versus-pacbio-error-profile-comparison
description: Use when evaluating how much a consensus-generation step (e.g., SPOA
  partial order alignment) reduces raw read error on error-prone third-generation
  sequencing data, or when comparing substitution/insertion/deletion error profiles
  between ONT and PacBio synthetic or real reads.
when_to_use_negative:
- Input reads are already high-accuracy short reads (Illumina) where consensus correction
  is unnecessary and the ONT/PB error profiles do not apply
- The goal is TR genotyping or motif resolution rather than measuring consensus error
  reduction
- Reads come from unassembled regions (centromeres, telomeres), where TRiCoLOR cannot
  identify repeats
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR REFER
  role: Computes haplotype-specific low-error consensus sequences via SPOA partial
    order alignment, whose error reduction is being quantified
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR
  role: Tool under evaluation; installed from source or Docker container for running
    the REFER consensus step
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:82d0110da9d482eb6ed557b9943f0a4b1e550e2733ff324a75695263794e8f21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ont-versus-pacbio-error-profile-comparison

## Summary

Quantifies and compares per-base sequencing error rates of ONT versus PacBio long reads before and after POA-based consensus correction, as done in TRiCoLOR's REFER evaluation where consensus computation reduced error by ~77% (ONT) and ~88% (PB).

## When to use

Use when evaluating how much a consensus-generation step (e.g., SPOA partial order alignment) reduces raw read error on error-prone third-generation sequencing data, or when comparing substitution/insertion/deletion error profiles between ONT and PacBio synthetic or real reads.

## When NOT to use

- Input reads are already high-accuracy short reads (Illumina) where consensus correction is unnecessary and the ONT/PB error profiles do not apply
- The goal is TR genotyping or motif resolution rather than measuring consensus error reduction
- Reads come from unassembled regions (centromeres, telomeres), where TRiCoLOR cannot identify repeats

## Inputs

- Raw error-prone long-read FASTQ files (ONT and PacBio, e.g., VISOR-simulated with mean read length 8,000 bp)
- Haplotype-specific consensus BAM files from TRiCoLOR REFER (SPOA + minimap2 realignment)
- Reference genome FASTA (e.g., GRCh38)

## Outputs

- Per-platform per-base error rates (substitutions, insertions, deletions) for raw and consensus reads
- Error reduction percentages for ONT and PB (expected ~77% and ~88% respectively)
- consensus_error_reduction.csv with columns: technology, raw error rate, consensus error rate, reduction percentage, reported reference value

## How to apply

Generate or obtain haplotype-specific consensus BAM files produced by TRiCoLOR REFER via SPOA partial order alignment and minimap2 realignment. Align both the raw simulated reads and the consensus sequences to the same reference (e.g., GRCh38 for VISOR-simulated ONT reads with mean length 8,000 bp and error profile substitution:insertion:deletion ~45:25:30, and PacBio reads with profile ~15:50:35) using minimap2. Compute the per-base error rate (substitutions, insertions, and deletions per aligned base) for each dataset via edit-distance-based comparison. Report error reduction as (raw error rate - consensus error rate) / raw error rate x 100, separately per platform, and compare against the reported ~77% (ONT) and ~88% (PB) values using a tolerance band of about +/-5 percentage points.

## Related tools

- **TRiCoLOR REFER** (Computes haplotype-specific low-error consensus sequences via SPOA partial order alignment, whose error reduction is being quantified) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR** (Tool under evaluation; installed from source or Docker container for running the REFER consensus step) — https://github.com/davidebolo1993/TRiCoLOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa ont_reads.fastq | samtools sort -o raw_ont.bam
```

## Evaluation signals

- Computed error reduction falls within +/-5 percentage points of ~77% for ONT and ~88% for PB
- Raw ONT error profile approximates substitution:insertion:deletion of ~45:25:30; PB profile approximates ~15:50:35
- consensus_error_reduction.csv contains all required columns (technology, raw error rate, consensus error rate, reduction percentage, reported reference value)
- Raw and consensus reads are aligned to the identical reference with the same aligner (minimap2) so error-rate differences reflect correction, not mapping differences

## Limitations

- Reported values are approximate (~77%, ~88%) and derived from simulated (VISOR) data; real-data reductions may differ
- Supplementary Note S3 / Supplementary Fig. S2 content is not included in the available text, so exact per-platform raw and consensus error rates cannot be independently verified
- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres), which bounds where consensus quality can be profiled

## Evidence

- [methods] SPOA achieves approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [results] We aligned the ONT FASTQ files to the human GRCh38 reference genome using minimap2: "We aligned the ONT FASTQ files to the human GRCh38 reference genome using minimap2"
- [readme] TRiCoLOR is available to be installed from source or as a Docker container: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
