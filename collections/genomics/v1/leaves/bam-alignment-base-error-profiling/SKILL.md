---
name: bam-alignment-base-error-profiling
description: Use when you need to measure how much a consensus-generation step (e.g.,
  SPOA/POA) lowers raw read error rates of ONT or PacBio long reads, or when benchmarking
  consensus accuracy on simulated VISOR reads against published reduction figures.
when_to_use_negative:
- Reads are already high-accuracy (e.g., Illumina or HiFi) — the ~77%/~88% reduction
  figures apply to error-prone third-generation ONT/PB reads
- The goal is tandem repeat genotyping itself (use the full TRiCoLOR SENSoR/REFER/profiling
  pipeline) rather than error-rate benchmarking of its consensus step
- Repeats or reads lie in unassembled genomic regions (e.g., centromeres, telomeres),
  where TRiCoLOR cannot be applied and the error model is undefined
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR REFER
  role: Generates haplotype-specific low-error consensus sequences via SPOA partial
    order alignment and minimap2 realignment, the consensus output whose error reduction
    is profiled
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: minimap2
  role: Aligns raw simulated reads and consensus sequences to the same reference (e.g.,
    GRCh38) for edit-distance-based per-base error computation; chosen for favorable
    speed and mapping accuracy on synthetic data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:767b4c599e01c42d7c9025648e761adf5afe502c22e5be9a0ae793cf5100b438
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bam-alignment-base-error-profiling

## Summary

Quantify per-base sequencing error (substitutions, insertions, deletions) in error-prone long reads versus POA-derived consensus sequences, expressed as percentage error reduction. Grounded in TRiCoLOR's REFER module, which uses SPOA partial order alignment to achieve ~77% (ONT) and ~88% (PacBio) error reduction.

## When to use

Use when you need to measure how much a consensus-generation step (e.g., SPOA/POA) lowers raw read error rates of ONT or PacBio long reads, or when benchmarking consensus accuracy on simulated VISOR reads against published reduction figures.

## When NOT to use

- Reads are already high-accuracy (e.g., Illumina or HiFi) — the ~77%/~88% reduction figures apply to error-prone third-generation ONT/PB reads
- The goal is tandem repeat genotyping itself (use the full TRiCoLOR SENSoR/REFER/profiling pipeline) rather than error-rate benchmarking of its consensus step
- Repeats or reads lie in unassembled genomic regions (e.g., centromeres, telomeres), where TRiCoLOR cannot be applied and the error model is undefined

## Inputs

- Raw error-prone long-read FASTQ/BAM files (ONT or PacBio, e.g., VISOR-simulated ONT reads mean length 8,000 bp)
- Haplotype-specific consensus BAM files from TRiCoLOR REFER (SPOA + minimap2 realignment)
- Reference genome alignment (e.g., GRCh38) produced by minimap2

## Outputs

- Per-base error rates (substitutions, insertions, deletions per aligned base) for raw and consensus datasets
- consensus_error_reduction.csv with columns: technology, raw error rate, consensus error rate, reduction percentage, reported reference value

## How to apply

Generate or obtain haplotype-specific consensus BAMs produced by TRiCoLOR's REFER module, which uses SPOA partial order alignment plus minimap2 realignment to compute consensus sequences with an approximate error reduction of ~77% for ONT and ~88% for PB. Align raw simulated reads (e.g., VISOR-generated ONT reads of mean length 8,000 bp with substitution:insertion:deletion profile ~45:25:30, and PacBio reads at ~15:50:35) and the consensus sequences to the same reference (e.g., GRCh38) with minimap2, which compared favorably to alternative aligners on speed and mapping accuracy. Compute per-base error rates (substitutions, insertions, deletions per aligned base) for raw and consensus datasets via edit-distance-based comparison. Calculate reduction percentage as (raw error rate - consensus error rate) / raw error rate x 100, separately per platform, and compare to the reported ~77% (ONT) and ~88% (PB) values from Supplementary Note S3 / Supplementary Fig. S2.

## Related tools

- **TRiCoLOR REFER** (Generates haplotype-specific low-error consensus sequences via SPOA partial order alignment and minimap2 realignment, the consensus output whose error reduction is profiled) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **minimap2** (Aligns raw simulated reads and consensus sequences to the same reference (e.g., GRCh38) for edit-distance-based per-base error computation; chosen for favorable speed and mapping accuracy on synthetic data)

## Examples

```
docker pull davidebolo1993/tricolor
```

## Evaluation signals

- Computed error reduction falls within ~+/-5 percentage points of ~77% for ONT and ~88% for PacBio simulated reads
- consensus_error_reduction.csv contains all required columns: technology, raw error rate, consensus error rate, reduction percentage, reported reference value
- Reduction percentages computed separately per platform match the ONT (~45:25:30 substitution:insertion:deletion) and PB (~15:50:35) error profiles
- Consensus BAMs are haplotype-specific and were generated by SPOA partial order alignment within the REFER module, not by ad-hoc consensus tools

## Limitations

- Reported values are approximate (~77% ONT, ~88% PB) and were derived on VISOR-simulated reads; exact empirical recomputation may vary with simulation parameters
- Supporting material (Supplementary Note S3 / Supplementary Fig. S2) is referenced but not included in the main text, limiting exact protocol reproduction
- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres), so error profiling there is out of scope

## Evidence

- [methods] SPOA achieves approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [intro] POA used for haplotype-specific consensus: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] minimap2 chosen for alignment benchmark: "minimap2 [19], which compared favorably to alternative aligners on syntethic data, in terms of both speed and mapping accuracy"
- [readme] Tool obtainable as container for reuse: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
