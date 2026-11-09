---
name: feature-rank-verification-against-published-table
description: Use when reproducing a published feature-correlation table (e.g., Table
  1 of a ΔΨ prediction paper) or when validating that individual splicing features
  on a benchmark training set match reported PCC values before feature selection or
  model rebuilding.
when_to_use_negative:
- The goal is model prediction of ΔΨ for new variants rather than feature-ranking
  verification — use the full PredPSI-SVR pipeline (main.sh with a VCF and optional
  -p PSI file) instead.
- The input is already a reduced feature table (e.g., the eight features from greedy
  feature selection) — recomputing the 42-feature ranking adds no information.
- Variants come from the blind test set or filtered sets (e.g., the 53 mutations remaining
  after removing |ΔΨ| < 2 standard deviations) — the reported Table 1 values apply
  specifically to the TR957 training set.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3320
tools:
- name: PredPSI-SVR
  role: Source of the TR957 training dataset, the 42-feature extraction pipeline,
    and the published Table 1 ranking being verified
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: Python 3 + numpy
  role: Compute Pearson correlation coefficients between each splicing feature and
    experimental ΔΨ
- name: Perl
  role: Required in PATH by the PredPSI-SVR environment from which features and training
    data are derived
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:848ba30088bd024ab86a2b746bec120d29903d96f079d2f05b0aaf17ec655bd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - Perl
  - PredPSI-SVR
  - Python 3
  - Python 3 + numpy
  - numpy
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# feature-rank-verification-against-published-table

## Summary

Reproduces and verifies the per-feature Pearson correlation (PCC) ranking of the 42 splicing-related features against experimental ΔΨ on the vex-seq training set TR957, checking that ΔMES5 is the top feature (PCC = 0.402) and that exactly nine features exceed |PCC| > 0.1 as reported in Table 1 of the PredPSI-SVR paper.

## When to use

Use when reproducing a published feature-correlation table (e.g., Table 1 of a ΔΨ prediction paper) or when validating that individual splicing features on a benchmark training set match reported PCC values before feature selection or model rebuilding.

## When NOT to use

- The goal is model prediction of ΔΨ for new variants rather than feature-ranking verification — use the full PredPSI-SVR pipeline (main.sh with a VCF and optional -p PSI file) instead.
- The input is already a reduced feature table (e.g., the eight features from greedy feature selection) — recomputing the 42-feature ranking adds no information.
- Variants come from the blind test set or filtered sets (e.g., the 53 mutations remaining after removing |ΔΨ| < 2 standard deviations) — the reported Table 1 values apply specifically to the TR957 training set.

## Inputs

- TR957 vex-seq challenge training dataset with 42 extracted splicing-related features per variant
- Experimental ΔΨ (delta percent-spliced-in) values for each TR957 variant
- Table 1 of the PredPSI-SVR paper (expected feature names, PCC values, and the |PCC| > 0.1 subset)

## Outputs

- feature_pcc_ranking.csv with per-feature PCC values and ranks
- Ranked list of the nine features with |PCC| > 0.1, led by ΔMES5 (PCC = 0.402)

## How to apply

Load the vex-seq challenge training dataset (TR957) from the 5th Critical Assessment of Genome Interpretation containing extracted splicing features and experimental ΔΨ values. Using Python 3 with the numpy package (the same interpreter stack PredPSI-SVR requires), compute the Pearson correlation coefficient between each of the 42 comprehensive splicing-related features — sequence, conservation, splice site (e.g., MaxEntScan scores), regulatory element, and mRNA structure features — and experimental ΔΨ. Rank all 42 features by absolute PCC and select the subset with |PCC| > 0.1. Verify that ΔMES5 ranks first with PCC = 0.402 and that exactly nine features pass the |PCC| = 0.1 threshold, matching Table 1. Write per-feature PCC values and ranks to feature_pcc_ranking.csv for downstream greedy feature selection, which in the paper reduced the 42 features to the eight most contributing ones.

## Related tools

- **PredPSI-SVR** (Source of the TR957 training dataset, the 42-feature extraction pipeline, and the published Table 1 ranking being verified) — https://github.com/chenkenbio/PredPSI-SVR
- **Python 3 + numpy** (Compute Pearson correlation coefficients between each splicing feature and experimental ΔΨ)
- **Perl** (Required in PATH by the PredPSI-SVR environment from which features and training data are derived)

## Examples

```
python3 -c "import numpy as np, csv; d=np.genfromtxt('TR957_features.tsv',names=True); pcc={f:np.corrcoef(d[f],d['dPsi'])[0,1] for f in d.dtype.names if f!='dPsi'}; [print(f,round(v,3)) for f,v in sorted(pcc.items(),key=lambda x:-abs(x[1]))]" > feature_pcc_ranking.csv
```

## Evaluation signals

- Computed PCC of ΔMES5 equals the reported value 0.402 on TR957
- Exactly nine features have |PCC| > 0.1, and their identities match Table 1 of the paper
- All 42 features yield finite PCC values with no NaNs from missing feature columns
- Ranking by absolute PCC places ΔMES5 first and reproduces the relative order of the remaining Table 1 features
- feature_pcc_ranking.csv contains one row per feature (42 rows) with rank and PCC columns populated

## Limitations

- The reported ranking is specific to the small TR957 vex-seq training set; small sample size prevented use of more powerful algorithms like deep learning and limited feature coverage.
- PredPSI-SVR does not effectively include branch point sequence or pre-mRNA secondary structure features, so these cannot be verified against Table 1.
- Correlations are computed against experimental wild-type Ψ-dependent ΔΨ values; without experimental Ψ the paper reports PCCs drop by about 0.1.

## Evidence
