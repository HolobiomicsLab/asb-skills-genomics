---
name: sv-call-vs-truth-matching
description: Use when you have SAM/BAM alignments from multiple long-read mappers
  over a simulated benchmark with a known ground-truth SV set, and need to rank mapper-caller
  combinations by inversion detection sensitivity and precision.
when_to_use_negative:
- The dataset is real sequencing data (e.g. NA12878) without a known ground truth
  — there is no truth set to match against, so sensitivity/precision cannot be computed
- The analysis goal is raw mapping alignment quality rather than downstream SV calling
  comparison — use the mapping benchmark metrics instead
- The variant type of interest is not inversion SVs, since this comparison was designed
  and validated for inversion detection
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: invMap
  role: Long-read mapper whose SAM alignments are one input arm of the comparison;
    anchors its chaining scoring to discover potential inversions
  repo: https://github.com/zhang134/invMap.git
- name: SVision
  role: SV caller run on each mapper's SAM/BAM alignment file to produce per-mapper
    inversion call sets
- name: SURVIVOR
  role: Used to derive the ground-truth inversion set and to evaluate/match caller
    output against that truth
- name: Winnowmap2
  role: Competing long-read mapper whose alignments feed the same SVision + SURVIVOR
    pipeline for the sensitivity comparison
- name: minimap2
  role: Competing long-read mapper whose alignments feed the same SVision + SURVIVOR
    pipeline for the sensitivity comparison
- name: NGMLR
  role: Competing long-read mapper whose alignments feed the same SVision + SURVIVOR
    pipeline for the sensitivity comparison
- name: lra
  role: Competing long-read mapper whose alignments feed the same SVision + SURVIVOR
    pipeline for the sensitivity comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:265c02e07b07a5a5960cbbd92f45bbf2b76f1b8f7ff319e4fdca7fe2b69df9d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sv-call-vs-truth-matching

## Summary

Run an SV caller (SVision) on each mapper's long-read alignments and match the resulting inversion call sets against a SURVIVOR-derived simulated ground truth to compute per-mapper total calls, correct calls, sensitivity, and precision. This is how the invMap paper builds its Table 2 comparison showing invMap-SVision achieves 125 calls with 82 correct and a 2.5%-15.49% sensitivity gain.

## When to use

Use when you have SAM/BAM alignments from multiple long-read mappers over a simulated benchmark with a known ground-truth SV set, and need to rank mapper-caller combinations by inversion detection sensitivity and precision.

## When NOT to use

- The dataset is real sequencing data (e.g. NA12878) without a known ground truth — there is no truth set to match against, so sensitivity/precision cannot be computed
- The analysis goal is raw mapping alignment quality rather than downstream SV calling comparison — use the mapping benchmark metrics instead
- The variant type of interest is not inversion SVs, since this comparison was designed and validated for inversion detection

## Inputs

- Per-mapper long-read alignment files (SAM/BAM) from the inversion benchmark, e.g. invMap_aligned.sam
- SURVIVOR-derived ground-truth inversion call set for the simulated benchmark genomes
- Mapper identity labels (invMap, Winnowmap2, minimap2, NGMLR, lra) attached to each alignment file

## Outputs

- Per-mapper VCF files of SVision inversion calls
- table2_sv_evaluation.csv with per-mapper total calls, correct calls, sensitivity, precision, and invMap sensitivity improvement

## How to apply

Collect the SAM/BAM alignment files produced by each mapper (invMap, Winnowmap2, minimap2, NGMLR, lra) in the inversion benchmark, attaching mapper identity to each file. Run SVision independently on each mapper's alignment to call inversion structural variants, keeping one call set per mapper. Load the SURVIVOR-derived ground-truth inversion set for the simulated benchmark genomes, then match each mapper's SVision calls against it to count total calls and correct calls per mapper. Compute sensitivity and precision per mapper and the relative sensitivity improvement of the invMap-SVision combination over competing mapper-caller pipelines. Write per-mapper call sets as VCF files and compile statistics into table2_sv_evaluation.csv, checking that the invMap-SVision pipeline reproduces 125 inversion calls with 82 correct calls. The rationale is that long PacBio/Oxford Nanopore reads more frequently span inversion breakpoints, so mapping quality directly drives downstream SV calling accuracy.

## Related tools

- **invMap** (Long-read mapper whose SAM alignments are one input arm of the comparison; anchors its chaining scoring to discover potential inversions) — https://github.com/zhang134/invMap.git
- **SVision** (SV caller run on each mapper's SAM/BAM alignment file to produce per-mapper inversion call sets)
- **SURVIVOR** (Used to derive the ground-truth inversion set and to evaluate/match caller output against that truth)
- **Winnowmap2** (Competing long-read mapper whose alignments feed the same SVision + SURVIVOR pipeline for the sensitivity comparison)
- **minimap2** (Competing long-read mapper whose alignments feed the same SVision + SURVIVOR pipeline for the sensitivity comparison)
- **NGMLR** (Competing long-read mapper whose alignments feed the same SVision + SURVIVOR pipeline for the sensitivity comparison)
- **lra** (Competing long-read mapper whose alignments feed the same SVision + SURVIVOR pipeline for the sensitivity comparison)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The invMap-SVision pipeline calls exactly 125 inversions in total with 82 correct calls
- invMap-SVision sensitivity improvement over competing mappers falls within the 2.5%-15.49% range
- Every mapper-caller combination yields a per-mapper VCF plus counts of total calls and correct calls against the SURVIVOR truth set
- Per-mapper sensitivity and precision values are computed for all five mappers and compiled into table2_sv_evaluation.csv without missing rows
- Alignment inputs are valid SAM format (e.g. invMap_aligned.sam) and mapper identity is preserved for every call set

## Limitations

- Sensitivity/precision evaluation depends on simulated genomes with a known ground truth; it cannot be applied to real datasets like NA12878 where truth is unavailable
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Results reflect the specific caller (SVision) and truth-set derivation (SURVIVOR); other caller-truth combinations may not reproduce the 125/82 figures

## Evidence

- [other] The invMap-SVision pipeline calls 125 inversions in total with 82 correct calls, achieving the best recover sensitivity with a 2.5%-15.49% improvement: [evidence span withheld — anchor status: not_found]
- [abstract] invMap chaining-based inversion discovery: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] SAM output of invMap: "The final mapping result is invMap_aligned.sam in SAM format."
- [abstract] long reads spanning SV breakpoints rationale: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
