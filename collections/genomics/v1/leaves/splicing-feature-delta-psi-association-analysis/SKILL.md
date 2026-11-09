---
name: splicing-feature-delta-psi-association-analysis
description: Use when you have a table of extracted splicing features (e.g. the 42
  PredPSI-SVR features) paired with experimental ΔΨ values and need to rank individual
  features by univariate association strength.
when_to_use_negative:
- The goal is multivariate predictive modeling or building the final PredPSI-SVR model
  — use the greedy feature selection workflow (eight selected features) instead of
  univariate PCC ranking
- The 42 splicing features have not yet been extracted from the exon/flanking sequences
  (e.g. raw VCF input without PredPSI-SVR feature annotation)
- Working on the blind test set or another dataset where the |PCC| > 0.1 nine-feature
  set from Table 1 is not the expected reference
tools:
- name: Python 3
  role: Runtime for computing per-feature Pearson correlations against ΔΨ
- name: numpy
  role: Python package used to compute the Pearson correlation coefficients
- name: MaxEntScan
  role: Source of the splice-site features (e.g. ΔMES5) that achieve the highest correlations
    with ΔΨ
- name: PredPSI-SVR
  role: Pipeline that produced the 42 splicing-related features and the TR957 training
    set analyzed here
  repo: https://github.com/chenkenbio/PredPSI-SVR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:a9d1d80a40f7293376cefd21f2a8e1d0fe9cfa1cc685e79c0a1097056807878c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - Python 3
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# splicing-feature-delta-psi-association-analysis

## Summary

Computes per-feature Pearson correlation coefficients (PCC) between each of the 42 splicing-related features extracted from exons and their flanking regions and the experimental ΔΨ values on the vex-seq training set TR957, then ranks features by |PCC| to identify the features most individually associated with splicing change.

## When to use

Use when you have a table of extracted splicing features (e.g. the 42 PredPSI-SVR features) paired with experimental ΔΨ values and need to rank individual features by univariate association strength.

## When NOT to use

- The goal is multivariate predictive modeling or building the final PredPSI-SVR model — use the greedy feature selection workflow (eight selected features) instead of univariate PCC ranking
- The 42 splicing features have not yet been extracted from the exon/flanking sequences (e.g. raw VCF input without PredPSI-SVR feature annotation)
- Working on the blind test set or another dataset where the |PCC| > 0.1 nine-feature set from Table 1 is not the expected reference

## Inputs

- TR957 vex-seq challenge training dataset (5th Critical Assessment of Genome Interpretation) with experimental ΔΨ values
- Per-variant matrix of the 42 extracted splicing-related features (sequence, conservation, splice site, regulatory element, and mRNA secondary structure features)

## Outputs

- feature_pcc_ranking.csv containing each feature's PCC against ΔΨ and its rank
- Subset of features with |PCC| > 0.1 (expected to contain nine features including ΔMES5, PCC = 0.402)

## How to apply

Load the TR957 training set (the vex-seq challenge training dataset from CAGI 5) containing the extracted splicing features and experimental ΔΨ values. Using Python 3 with the numpy package, compute the Pearson correlation coefficient of each of the 42 features against ΔΨ individually. Rank all features by absolute PCC and select the subset with |PCC| > 0.1 as candidate strongly associated features. Compare against the paper's Table 1, where splicing-site features (e.g. those computed by MaxEntScan such as ΔMES5) rank highest — ΔMES5 is the single most correlated feature with PCC = 0.402 and exactly nine features exceed the |PCC| = 0.1 threshold. Note this univariate ranking is diagnostic only; the paper used a separate greedy feature selection algorithm (yielding eight features) to build the final SVR model.

## Related tools

- **Python 3** (Runtime for computing per-feature Pearson correlations against ΔΨ)
- **numpy** (Python package used to compute the Pearson correlation coefficients)
- **MaxEntScan** (Source of the splice-site features (e.g. ΔMES5) that achieve the highest correlations with ΔΨ)
- **PredPSI-SVR** (Pipeline that produced the 42 splicing-related features and the TR957 training set analyzed here) — https://github.com/chenkenbio/PredPSI-SVR

## Examples

```
python3 -c "import numpy as np; pcc = np.corrcoef(feature_values, delta_psi)[0,1]; print(pcc)"
```

## Evaluation signals

- The computed PCC of ΔMES5 against ΔΨ on TR957 equals the reported value 0.402
- Exactly nine features exceed the |PCC| = 0.1 threshold, matching Table 1 of the paper
- The rank ordering of features in feature_pcc_ranking.csv reproduces Table 1, with splice-site features (MaxEntScan-derived) at the top
- PCC values fall in [-1, 1] and are computed per feature (42 rows) with no missing feature entries
- Cross-check context: the full PredPSI-SVR model built on selected features reaches PCC 0.570 in 10-fold CV, so any feature set implying far higher univariate correlations should be flagged

## Limitations

- Univariate PCC ranking does not capture feature interactions or redundancy; the paper found only eight features contributing most to prediction via greedy feature selection, which differs from the nine |PCC| > 0.1 features
- The small number of TR957 samples limits statistical power and led the authors to avoid more powerful algorithms like deep learning
- PredPSI-SVR features do not include branch point sequences or pre-mRNA secondary structure effectively due to the small sample size

## Evidence
