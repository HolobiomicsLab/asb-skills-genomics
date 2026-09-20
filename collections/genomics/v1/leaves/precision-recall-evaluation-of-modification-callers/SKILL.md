---
name: precision-recall-evaluation-of-modification-callers
description: Use when you have reads with per-read ground-truth modification labels
  (e.g. synthetic oligos where even-cycle GGACU positions are known modified) and
  need to compare modification callers by read-level precision-recall curves and PR
  AUC.
when_to_use_negative:
- The benchmark exposes only ensemble-level modification status with no per-read ground
  truth (e.g. natural mRNA with approximate labels), so read-level PR AUC cannot be
  computed
- Input reads have already been filtered/aligned or modification probabilities have
  already been computed — proceed directly to the thresholding and PR calculation
  step
- Evaluation target is site-level stoichiometry accuracy (e.g. comparison against
  GLORI measurements with min coverage 50), not single-molecule caller discrimination
tools:
- name: RODAN
  role: Backbone basecaller/feature-extraction model (backbone.torch) used to basecall
    TEST2 fast5 files before modification calling
  repo: https://github.com/biodlab/RODAN
- name: minimap2
  role: Aligns basecalled TEST2 reads to reference sequences with the splice/uf preset
    and -k14
- name: samtools
  role: Filters BAM with view -q50 to remove reads with mapping quality below 50,
    then sorts and indexes
- name: mAFiA
  role: Primary method under test; produces per-read m6A probabilities with --min_coverage
    50, --max_num_reads 1000, --mod_prob_thresh 0.5
  repo: https://github.com/dieterich-lab/mAFiA
- name: CHEUI
  role: Comparison method run on the same TEST2 dataset for the PR AUC benchmark
- name: m6Anet
  role: Comparison method run on the same TEST2 dataset for the PR AUC benchmark
- name: sklearn
  role: Python module used to compute precision-recall curves and AUC values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:8682b3dac181b43ae1d975b5ec7111b81991480271089933ddbcdbbd61012875
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - CHEUI
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2
  - samtools
  - sklearn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# precision-recall-evaluation-of-modification-callers

## Summary

Benchmark dRNA-Seq m6A detection methods (mAFiA, CHEUI, m6Anet) at single-read resolution by computing precision-recall curves and PR AUC against known true-positive modifications on the TEST2 periodic-repeat synthetic benchmark, where reads carry a phase-shifted m6A pattern invisible at the aggregate site level.

## When to use

Use when you have reads with per-read ground-truth modification labels (e.g. synthetic oligos where even-cycle GGACU positions are known modified) and need to compare modification callers by read-level precision-recall curves and PR AUC.

## When NOT to use

- The benchmark exposes only ensemble-level modification status with no per-read ground truth (e.g. natural mRNA with approximate labels), so read-level PR AUC cannot be computed
- Input reads have already been filtered/aligned or modification probabilities have already been computed — proceed directly to the thresholding and PR calculation step
- Evaluation target is site-level stoichiometry accuracy (e.g. comparison against GLORI measurements with min coverage 50), not single-molecule caller discrimination

## Inputs

- TEST2 periodic-repeat synthetic dRNA-Seq fast5 files
- TEST2 reference sequences for alignment
- Known per-read ground-truth labels (even-cycle GGACU positions are m6A-modified)
- Pretrained models: RODAN-based backbone.torch and mAFiA pickled logistic regression classifiers

## Outputs

- Per-read modification probabilities P(m6A) from each caller (mAFiA, CHEUI, m6Anet)
- Precision-recall curves per tool at even-cycle GGACU positions (P threshold 0.5)
- Per-tool PR AUC values aggregated into pr_auc_comparison.csv

## How to apply

Basecall the TEST2 dRNA-Seq fast5 files with the RODAN-based backbone model (backbone.torch) and align the reads with minimap2 (splice/uf preset, -k14), then filter with samtools view -q50 to remove reads with mapping quality below 50. Run mAFiA (test_mAFiA) on the filtered BAM with --min_coverage 50, --max_num_reads 1000, --backbone_model_path backbone.torch, --classifier_model_dir classifiers, and --mod_prob_thresh 0.5 to obtain per-read modification probabilities. Run CHEUI and m6Anet on the same TEST2 dataset as comparison methods, then classify each read at the even-cycle GGACU positions as modified/unmodified at a P threshold of 0.5. Compute precision-recall curves and AUC against the known even-cycle true positives with sklearn and aggregate the per-tool PR AUC values into pr_auc_comparison.csv. The rationale is that TEST2 appears ~50% modified at the aggregate site level, so only read-resolved discrimination exposes true caller performance.

## Related tools

- **RODAN** (Backbone basecaller/feature-extraction model (backbone.torch) used to basecall TEST2 fast5 files before modification calling) — https://github.com/biodlab/RODAN
- **minimap2** (Aligns basecalled TEST2 reads to reference sequences with the splice/uf preset and -k14)
- **samtools** (Filters BAM with view -q50 to remove reads with mapping quality below 50, then sorts and indexes)
- **mAFiA** (Primary method under test; produces per-read m6A probabilities with --min_coverage 50, --max_num_reads 1000, --mod_prob_thresh 0.5) — https://github.com/dieterich-lab/mAFiA
- **CHEUI** (Comparison method run on the same TEST2 dataset for the PR AUC benchmark)
- **m6Anet** (Comparison method run on the same TEST2 dataset for the PR AUC benchmark)
- **sklearn** (Python module used to compute precision-recall curves and AUC values)

## Examples

```
minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}
```

## Evaluation signals

- PR AUC for mAFiA on TEST2 should substantially exceed that of CHEUI and m6Anet, reflecting mAFiA's resolution of the read-level even-cycle pattern (Fig. 2f)
- Each read classified at even-cycle GGACU positions using a P threshold of 0.5; sanity-check that modified and unmodified class sizes match the known ground truth
- The filtered BAM should contain only reads with mapping quality >= 50 (samtools view -q50)
- mAFiA run parameters must match the benchmark configuration: --min_coverage 50, --max_num_reads 1000, --mod_prob_thresh 0.5
- pr_auc_comparison.csv should contain one PR AUC value per tool (mAFiA, CHEUI, m6Anet) computed against the same even-cycle true-positive set

## Limitations

- Aggregate site-level measurements on TEST2 cannot distinguish the underlying pattern — all GGACU sites appear ~50% modified, so site-level metrics are uninformative for this benchmark
- Comparison methods CHEUI and m6Anet show considerable performance deterioration on this read-resolved task, so cross-tool comparisons at site level will not reflect read-level differences
- Reads with mapping quality below 50 are excluded; low-coverage sites (coverage < 50) are not evaluated
- The binary classification threshold is fixed at P(m6A) = 0.5 for the PR computation, so callers with different score calibrations are compared only through the resulting ranking

## Evidence

- [results] On the TEST2 benchmark, mAFiA resolves the underlying read-level m6A pattern, whereas analogous calculations with CHEUI and m6Anet show considerable deterioration: [evidence span withheld — anchor status: not_found]
- [results] Read-level P(m6A) threshold of 0.5: [evidence span withheld — anchor status: not_found]
- [results] mapq >= 50 filter: "Reads with mapq below 50 are filtered out."
- [readme] minimap2/samtools alignment pipeline: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}"
