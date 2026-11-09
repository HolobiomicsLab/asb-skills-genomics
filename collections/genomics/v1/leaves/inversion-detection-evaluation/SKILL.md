---
name: inversion-detection-evaluation
description: Use when comparing inversion-detection performance of competing long-read
  mappers (invMap, Winnowmap2, minimap2, NGMLR, lra) on simulated benchmark genomes
  with known ground-truth inversions, or when compiling per-mapper SV call statistics
  like sensitivity and precision.
when_to_use_negative:
- Input is a real human sequencing dataset (e.g. NA12878) rather than a simulated
  benchmark with SURVIVOR-derived ground truth — no reliable sensitivity/precision
  can be computed in that setting.
- The mapper alignments have not yet been produced (no SAM/BAM from the mapping benchmark
  step); the SV-calling step depends on per-mapper alignment files.
- Evaluating variant types other than inversion SVs, since this workflow is scoped
  to inversion structural variant calling and recovery sensitivity.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: invMap
  role: Long-read mapper whose SAM alignments serve as the invMap arm of the comparison;
    produces the best-performing pipeline (invMap-SVision) with 125 inversion calls,
    82 correct
  repo: https://github.com/zhang134/invMap.git
- name: SVision
  role: SV caller run on each mapper's alignment file to call inversion structural
    variants, producing per-mapper call sets
- name: SURVIVOR
  role: Used to derive the ground-truth inversion set for the simulated benchmark
    genomes used to score calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:8b9a7a72828058f02f02925a2b872bf0668160671c79399f6fe0c8424c59aa8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - SURVIVOR
  - SVision
  - invMap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inversion-detection-evaluation

## Summary

Evaluates long-read mapper pipelines for inversion structural variant (SV) detection by running SVision on each mapper's SAM/BAM alignments and matching calls against a SURVIVOR-derived simulated ground truth, as in the invMap Table 2 benchmark where invMap-SVision achieved 125 total calls with 82 correct and a 2.5%-15.49% sensitivity improvement.

## When to use

Use when comparing inversion-detection performance of competing long-read mappers (invMap, Winnowmap2, minimap2, NGMLR, lra) on simulated benchmark genomes with known ground-truth inversions, or when compiling per-mapper SV call statistics like sensitivity and precision.

## When NOT to use

- Input is a real human sequencing dataset (e.g. NA12878) rather than a simulated benchmark with SURVIVOR-derived ground truth — no reliable sensitivity/precision can be computed in that setting.
- The mapper alignments have not yet been produced (no SAM/BAM from the mapping benchmark step); the SV-calling step depends on per-mapper alignment files.
- Evaluating variant types other than inversion SVs, since this workflow is scoped to inversion structural variant calling and recovery sensitivity.

## Inputs

- Per-mapper alignment files (SAM/BAM) from the inversion benchmark mapping step (invMap and competing long-read mappers)
- SURVIVOR-derived ground-truth inversion set for the simulated benchmark genomes

## Outputs

- Per-mapper SVision inversion call sets written as VCF files
- Comparison statistics table table2_sv_evaluation.csv with total calls, correct calls, sensitivity, precision, and relative sensitivity improvement per mapper

## How to apply

Collect the alignment (SAM/BAM) files produced in the inversion benchmark mapping step for each mapper, keeping mapper identity attached to each call set. Run SVision on each mapper's alignment file to call inversion SVs, then load the SURVIVOR-derived ground-truth inversion set for the simulated benchmark genomes. Match each mapper's SVision calls against the ground truth to count total calls and correct calls per mapper, then compute sensitivity and precision per mapper and the relative sensitivity improvement of invMap-SVision over competing mapper-caller combinations. Write per-mapper call sets as VCF files and compile comparison statistics into table2_sv_evaluation.csv; success is reproducing 125 total inversion calls with 82 correct calls for invMap-SVision and confirming the 2.5%-15.49% sensitivity gain over Winnowmap2, minimap2, NGMLR and lra.

## Related tools

- **invMap** (Long-read mapper whose SAM alignments serve as the invMap arm of the comparison; produces the best-performing pipeline (invMap-SVision) with 125 inversion calls, 82 correct) — https://github.com/zhang134/invMap.git
- **SVision** (SV caller run on each mapper's alignment file to call inversion structural variants, producing per-mapper call sets)
- **SURVIVOR** (Used to derive the ground-truth inversion set for the simulated benchmark genomes used to score calls)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap-SVision call set reproduces 125 total inversion calls with 82 correct calls
- Relative recover-sensitivity improvement of invMap-SVision over Winnowmap2, minimap2, NGMLR and lra falls in the reported 2.5%-15.49% range
- table2_sv_evaluation.csv contains per-mapper rows with total calls, correct calls, sensitivity, and precision
- Per-mapper outputs are valid VCF files with mapper identity attached to each call set
- Call-to-ground-truth matching counts total calls and correct calls consistently across mappers

## Limitations

- Evaluation is restricted to simulated benchmark genomes with SURVIVOR-derived ground truth; on real data such as the NA12878 human dataset only more candidate variant calls can be counted, not correct-call sensitivity.
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which may constrain re-running the mapping step.
- Results depend on the specific mapper-caller combination; sensitivity improvements are relative to Winnowmap2, minimap2, NGMLR and lra in this benchmark.

## Evidence

- [results] invMap-SVision achieves best recover sensitivity: [evidence span withheld — anchor status: not_found]
- [abstract] inversion discovery rationale: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] SAM output command: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [abstract] simulated accuracy finding: [evidence span withheld — anchor status: not_found]
