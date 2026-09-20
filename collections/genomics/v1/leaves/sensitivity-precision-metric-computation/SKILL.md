---
name: sensitivity-precision-metric-computation
description: Use when mapper- or caller-predicted inversion SV call sets must be scored
  against a known simulated truth set to produce sensitivity/precision rows for a
  comparison table.
when_to_use_negative:
- The dataset contains no ground-truth truth set (e.g., purely real sequencing data
  without simulated SVs), so recovered-call classification against known inversions
  is impossible
- The variant calls are not inversion structural variants (e.g., deletions, duplications)
  — this metric workflow was defined specifically for inversion recovery evaluation
tools:
- name: invMap
  role: Long-read mapper whose alignment output supplies the predicted inversion calls
    scored for sensitivity and precision
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: Generates the simulated ground-truth inversion set and is the reference for
    classifying predicted calls as recovered true positives
- name: SVision
  role: SV caller producing per-mapper inversion call sets whose sensitivity and precision
    are compared in the Table 2 evaluation
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:ef4ade0bf95855b2f1ea48c58b127bd23523e31efa7b1a8e28c0fe5ca11b427f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - PBSIM2
  - SURVIVOR
  - SVision
  - invMap
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sensitivity-precision-metric-computation

## Summary

Computes recover sensitivity (recovered/predicted ratio) and precision for structural-variant callers by matching predicted inversion calls against a SURVIVOR-derived simulated ground truth, as used to benchmark invMap against Winnowmap2, minimap2, NGMLR, and lra.

## When to use

Use when mapper- or caller-predicted inversion SV call sets must be scored against a known simulated truth set to produce sensitivity/precision rows for a comparison table.

## When NOT to use

- The dataset contains no ground-truth truth set (e.g., purely real sequencing data without simulated SVs), so recovered-call classification against known inversions is impossible
- The variant calls are not inversion structural variants (e.g., deletions, duplications) — this metric workflow was defined specifically for inversion recovery evaluation

## Inputs

- Predicted inversion SV call sets per mapper (e.g., invMap 3606 predictions from simulated chr1 CLR reads)
- SURVIVOR-generated ground-truth inversion set (100 simulated inversions on GRCh37 chr1)
- SVision call sets from each mapper's SAM/BAM alignment files (Table 2 evaluation)
- Mapper identity labels for invMap, Winnowmap2, minimap2, NGMLR, and lra

## Outputs

- Per-mapper sensitivity (e.g., 92.06% for invMap) and precision (96.58%) values
- Recovery counts: predicted calls, correct/recovered calls (e.g., 3606 predicted, 3483 recovered; 125 calls with 82 correct for invMap-SVision)
- Relative sensitivity improvement statistics (2.5%-15.49% for invMap-SVision)
- Compiled comparison table (inversion_benchmark_table.csv, table2_sv_evaluation.csv) and per-mapper VCF files

## How to apply

Match each mapper's predicted inversion calls (e.g., from invMap, Winnowmap2, minimap2, NGMLR, lra alignments) against the 100 SURVIVOR-generated ground-truth inversions to count total calls and recovered (true-positive) calls. Compute sensitivity as the recovered/predicted ratio — in the reproduced Table 1, invMap reported 3606 predicted SVs, 3483 recovered SVs, and 3483/3606 = 92.06% sensitivity at 96.58% precision. For the SVision-based comparison, run SVision on each mapper's SAM/BAM alignments, match calls to the ground truth, and quantify the relative sensitivity gain of the invMap-SVision pipeline (125 total calls, 82 correct, a 2.5%-15.49% improvement). Compile per-mapper metrics into a CSV (e.g., inversion_benchmark_table.csv or table2_sv_evaluation.csv), attaching mapper identity to each call set throughout so downstream per-tool rows remain traceable. Accept stochastic-simulation tolerance when validating against published numbers.

## Related tools

- **invMap** (Long-read mapper whose alignment output supplies the predicted inversion calls scored for sensitivity and precision) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (Generates the simulated ground-truth inversion set and is the reference for classifying predicted calls as recovered true positives)
- **SVision** (SV caller producing per-mapper inversion call sets whose sensitivity and precision are compared in the Table 2 evaluation)

## Examples

```
python -c "sens=3483/3606*100; print(f'invMap sensitivity: {sens:.2f}%')"
```

## Evaluation signals

- invMap's reproduced Table 1 row reports exactly 3606 predicted SVs, 3483 recovered SVs, 92.06% sensitivity, and 96.58% precision (within stochastic-simulation tolerance)
- Sensitivity equals recovered/predicted, e.g., 3483/3606 = 92.06%; recompute the ratio from the call counts as a sanity check
- invMap-SVision reproduces 125 total inversion calls with 82 correct calls and a 2.5%-15.49% sensitivity improvement over competing mappers
- Every per-mapper metric row carries its mapper identity, and predicted calls are classified against the same 100 simulated ground-truth inversions
- Output CSV (inversion_benchmark_table.csv / table2_sv_evaluation.csv) contains both sensitivity and precision columns for all five mappers

## Limitations

- Reproduced values are subject to stochastic-simulation tolerance, not exact reproducibility of published numbers
- Metric accuracy depends entirely on the quality of the simulated ground truth (SURVIVOR simulateSV) and on correct classification of predicted calls as recovered true positives
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, constraining upstream alignment runs

## Evidence

- [other] invMap predicted 3606 inversions, correctly recovered 3483 of them, and achieved a recover sensitivity of 92.06%, outperforming Winnowmap2, minimap2, NGMLR, and lra.: [evidence span withheld — anchor status: not_found]
- [other] Compute sensitivity (recovered/predicted ratio, e.g. invMap: 3483/3606 = 92.06%) and precision for each mapper and assemble the Table 1 comparison into inversion_benchmark_table.csv.: [evidence span withheld — anchor status: not_found]
- [abstract] experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
