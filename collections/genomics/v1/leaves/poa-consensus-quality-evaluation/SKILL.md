---
name: poa-consensus-quality-evaluation
description: Use when you need to verify or reproduce the error-reduction benefit
  of POA-computed haplotype-specific consensus sequences from ONT or PacBio simulated
  reads before TR profiling.
when_to_use_negative:
- You need to profile tandem repeats themselves rather than quantify consensus accuracy
  — use TRiCoLOR's SENSoR/REFER TR-calling workflow instead
- Input reads are already high-accuracy (e.g., Illumina) where the POA consensus error-reduction
  benefit for error-prone third-generation reads does not apply
- Reads come from unassembled genomic regions (centromeres, telomeres), which TRiCoLOR
  cannot identify repeats in
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: TRiCoLOR REFER
  role: Computes haplotype-specific low-error consensus sequences via SPOA partial
    order alignment and minimap2 realignment; the resulting consensus error rate is
    the quantity being evaluated
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR
  role: Overall pipeline containing the REFER module and providing the consensus BAM
    outputs compared against raw reads
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:11df83444a639a0aeae238bf6e2717ece06bf8cc5b0684c742d5462fac4e4a54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# POA consensus quality evaluation

## Summary

Quantifies how much TRiCoLOR's SPOA-based partial order alignment consensus step reduces per-base sequencing error of error-prone long reads, benchmarked against the reported ~77% (ONT) and ~88% (PacBio) error reductions from Supplementary Note S3.

## When to use

Use when you need to verify or reproduce the error-reduction benefit of POA-computed haplotype-specific consensus sequences from ONT or PacBio simulated reads before TR profiling.

## When NOT to use

- You need to profile tandem repeats themselves rather than quantify consensus accuracy — use TRiCoLOR's SENSoR/REFER TR-calling workflow instead
- Input reads are already high-accuracy (e.g., Illumina) where the POA consensus error-reduction benefit for error-prone third-generation reads does not apply
- Reads come from unassembled genomic regions (centromeres, telomeres), which TRiCoLOR cannot identify repeats in

## Inputs

- Haplotype-specific consensus BAM files from TRiCoLOR REFER (SPOA + minimap2 realignment)
- Raw simulated error-prone long-read FASTQ files (VISOR ONT reads, mean 8,000 bp; PB reads)
- Reference genome FASTQ/FASTA for minimap2 alignment

## Outputs

- consensus_error_reduction.csv with columns: technology, raw error rate, consensus error rate, reduction percentage, reported reference value
- Per-base error rates (substitutions, insertions, deletions per aligned base) for raw and consensus datasets

## How to apply

Generate haplotype-specific consensus BAM files with TRiCoLOR REFER, which uses SPOA partial order alignment followed by minimap2 realignment, on simulated reads such as VISOR ONT reads (mean 8,000 bp; error profile substitution:insertion:deletion ~45:25:30) and PacBio reads (~15:50:35). Align both raw reads and consensus sequences to the same reference with minimap2 and compute per-base error rates (substitutions, insertions, deletions per aligned base) via edit-distance comparison. Compute the reduction percentage as (raw error rate - consensus error rate) / raw error rate x 100, separately per technology. Validate that computed reductions fall within ~5 percentage points of the reported ~77% (ONT) and ~88% (PB) values from Supplementary Note S3 / Supplementary Fig. S2, and write results to a CSV with technology, raw error rate, consensus error rate, reduction percentage, and reported reference value columns.

## Related tools

- **TRiCoLOR REFER** (Computes haplotype-specific low-error consensus sequences via SPOA partial order alignment and minimap2 realignment; the resulting consensus error rate is the quantity being evaluated) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR** (Overall pipeline containing the REFER module and providing the consensus BAM outputs compared against raw reads) — https://github.com/davidebolo1993/TRiCoLOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa raw_ont.fastq > raw.sam; minimap2 -ax map-ont GRCh38.fa consensus.bam > cons.sam; python compute_error_reduction.py --raw raw.sam --consensus cons.sam --out consensus_error_reduction.csv
```

## Evaluation signals

- Computed error reduction for ONT simulated reads falls within ~5 percentage points of the reported ~77%
- Computed error reduction for PB simulated reads falls within ~5 percentage points of the reported ~88%
- Consensus error rate is strictly lower than the raw read error rate for both ONT and PB
- Output CSV contains all five required columns (technology, raw error rate, consensus error rate, reduction percentage, reported reference value)
- Raw ONT error profile matches the expected substitution:insertion:deletion ratio of ~45:25:30 (PB ~15:50:35), confirming the simulation is appropriate

## Limitations

- The reported values are approximate (~77% ONT, ~88% PB) and were derived from synthetic/simulated reads, so exact reproduction depends on the read simulator and error profiles used
- Supplementary figures S1-S7 and notes S1-S8 (including Supplementary Note S3 / Fig. S2 with the reference values) are referenced but their full content is not included in the article text
- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres), so consensus evaluation is limited to assembled reference regions

## Evidence

- [methods] SPOA achieves approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [intro] TRiCoLOR exploits partial order alignment to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] minimap2 compared favorably to alternative aligners: "minimap2 [19], which compared favorably to alternative aligners on syntethic data, in terms of both speed and mapping accuracy"
- [readme] README citation and source links for TRiCoLOR: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
