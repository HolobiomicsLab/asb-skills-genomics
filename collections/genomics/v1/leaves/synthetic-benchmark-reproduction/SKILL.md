---
name: synthetic-benchmark-reproduction
description: Use when a reported tool comparison (e.g., recall/precision/F1 per variant
  class) must be independently reproduced from simulated inputs with a known truth
  set and explicitly stated validation thresholds.
when_to_use_negative:
- The evaluation target is real patient/population data without a known truth set
  (e.g., the HGSVC samples HG00514, HG00733, NA19240 used for validation-rate analysis
  instead of simulation)
- The claim concerns de novo discovery performance on unannotated regions rather than
  comparison of caller metrics against simulated ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR (SENSoR + REFER)
  role: Target long-read TR caller whose benchmark metrics are reproduced; SENSoR
    finds low-entropy regions, REFER builds SPOA haplotype consensus and genotypes
    TRs
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: NCRF
  role: Alternative alignment-based long-read TR caller run on the same simulated
    BAMs as the benchmark comparator
- name: SPOA
  role: Partial order alignment used within REFER to compute haplotype-specific consensus
    sequences with ~77% (ONT) and ~88% (PB) error reduction
- name: minimap2
  role: Aligner used to realign consensus sequences for TR detection; chosen for favorable
    speed and mapping accuracy on synthetic data
- name: VISOR
  role: Simulator used to generate haplotype-resolved ONT/PB BAMs embedding TR contractions
    and expansions as the benchmark ground truth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:16c6ff74517a7a8f99b85390671f5d6ef39832959c4d220ea9b37a5dff6d1170
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - NCRF
  - SPOA
  - TRiCoLOR
  - TRiCoLOR (SENSoR + REFER)
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - VISOR
  - minimap2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# synthetic-benchmark-reproduction

## Summary

Reproduce a published tool benchmark on synthetic data by regenerating simulated inputs with known truth, running the target tool and comparator(s) under the paper's default parameters, and recomputing per-class recall/precision/F1 to verify reported metrics within tolerance.

## When to use

Use when a reported tool comparison (e.g., recall/precision/F1 per variant class) must be independently reproduced from simulated inputs with a known truth set and explicitly stated validation thresholds.

## When NOT to use

- The evaluation target is real patient/population data without a known truth set (e.g., the HGSVC samples HG00514, HG00733, NA19240 used for validation-rate analysis instead of simulation)
- The claim concerns de novo discovery performance on unannotated regions rather than comparison of caller metrics against simulated ground truth

## Inputs

- VISOR-generated haplotype-resolved ONT and PacBio long-read BAM files (accuracy ~0.85-0.95, haplotype coverage 5-20x) with embedded TR contractions and expansions
- Simulated truth set of TR variants (class, position, motif, motif-count change)
- Published target metric values to reproduce (per-class recall/precision/F1 for TRiCoLOR and NCRF)

## Outputs

- TR calls in BED format (TRiCoLOR SENSoR candidate repetitive regions)
- TR calls in BCF format (TRiCoLOR REFER genotypes) and NCRF comparator calls
- benchmark_results.csv with per-class recall, precision, and F1 for both tools at 1- and 2-motif discrepancy thresholds

## How to apply

Generate synthetic inputs matching the paper's simulation design — here, VISOR-simulated haplotype-resolved ONT and PacBio BAMs with sequencing accuracy ~0.85-0.95 and haplotype coverage 5-20x, embedding TR contractions, small expansions (~7 motifs), and exceptionally long expansions (+200 motifs). Run the target tool (TRiCoLOR SENSoR with default Shannon-entropy settings: 20 bp non-overlapping sliding windows, entropy drop <= 1.23, >= 5 supporting reads, merging windows within 100 bp, followed by REFER with SPOA consensus and minimap2 realignment, motifs <= 6 bp, >= 5 perfect repeats, edit distance <= 1, minimum TR length >= 50 bp) and the comparator (NCRF) on the identical BAMs. Evaluate both against the simulated truth set at 1-motif and 2-motif discrepancy thresholds, verifying motif correctness for each simulated TR. Accept the reproduction only if recomputed metrics match reported values — TRiCoLOR R = 0.97 (contractions), 0.86 (expansions), 0.73 (long expansions); NCRF R = 0.87/0.74/0.53; F1 ~ 1 at 1-motif discrepancy; P ~ 1 and R ~ 1 at 2-motif discrepancy — within a small numerical tolerance.

## Related tools

- **TRiCoLOR (SENSoR + REFER)** (Target long-read TR caller whose benchmark metrics are reproduced; SENSoR finds low-entropy regions, REFER builds SPOA haplotype consensus and genotypes TRs) — https://github.com/davidebolo1993/TRiCoLOR
- **NCRF** (Alternative alignment-based long-read TR caller run on the same simulated BAMs as the benchmark comparator)
- **SPOA** (Partial order alignment used within REFER to compute haplotype-specific consensus sequences with ~77% (ONT) and ~88% (PB) error reduction)
- **minimap2** (Aligner used to realign consensus sequences for TR detection; chosen for favorable speed and mapping accuracy on synthetic data)
- **VISOR** (Simulator used to generate haplotype-resolved ONT/PB BAMs embedding TR contractions and expansions as the benchmark ground truth)

## Examples

```
docker run -v $(pwd):/data davidebolo1993/tricolor tricolor -b simulated_ont.bam -f reference.fasta -o /data/tricolor_calls
```

## Evaluation signals

- TRiCoLOR recall per class matches R = 0.97 (contractions), 0.86 (expansions), 0.73 (long expansions) within numerical tolerance
- NCRF recall per class matches R = 0.87 (contractions), 0.74 (expansions), 0.53 (long expansions) within tolerance
- F1 score is close to 1 at the 1-motif discrepancy threshold for both tools
- Precision ~ 1 and recall ~ 1 at the 2-motif discrepancy threshold
- Motifs are correctly identified for all simulated TRs (verification against the simulation truth set)

## Limitations

- Results depend on simulated read properties; the entropy threshold (1.23) and window size (20 bp) were empirically estimated and exact values for de novo identification of repetitive stretches are only described as ranges
- Benchmark performance on synthetic data does not guarantee the same behavior on real HGSVC whole-genome data
- TRiCoLOR itself cannot detect repeats in unassembled regions (centromeres, telomeres) or handle polyploid/haploid (e.g., human Y) chromosomes, bounding what the benchmark can cover

## Evidence

- [abstract] TRiCoLOR demonstrates excellent performance and improved sensitivity and specificity compared with alternative tools on synthetic data.: "TRiCoLOR demonstrates excellent performance and improved sensitivity and specificity compared with alternative tools on synthetic data."
- [methods] computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default).: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default)."
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [readme] TRiCoLOR is available to be installed from source or as a Docker container.: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
