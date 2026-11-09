---
name: vex-seq-training-data-analysis
description: Use when you have the vex-seq challenge training set TR957 with extracted
  splicing features and experimental ΔΨ values, and need to rank individual features
  by correlation with ΔΨ or validate the Table 1 feature ranking (ΔMES5, PCC = 0.402).
when_to_use_negative:
- The goal is to predict ΔΨ for new variants with the final PredPSI-SVR model rather
  than to analyze feature correlations on the training set
- You need greedy feature selection to derive the final eight-feature model, not the
  per-feature correlation ranking of all 42 features
- The data are from the vex-seq blind test set or another cohort rather than the TR957
  training set with experimental ΔΨ
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PredPSI-SVR
  role: Source of the TR957 training dataset pipeline and the 42 extracted splicing
    features whose PCC against ΔΨ is computed
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: numpy (Python 3)
  role: Computes the Pearson correlation coefficient of each of the 42 features against
    experimental ΔΨ
- name: Perl
  role: Runtime dependency of the PredPSI-SVR feature-extraction pipeline (e.g. MaxEntScan-based
    splice-site scoring) that must be in PATH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:88643e6e2946a32cdaeaf75091eeff926930f5c1a972396b63370e25eef5d53b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - Perl
  - PredPSI-SVR
  - Python 3
  - numpy
  - numpy (Python 3)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# vex-seq-training-data-analysis

## Summary

Reproduces the per-feature Pearson correlation (PCC) ranking of the 42 splicing-related features against experimental ΔΨ on the vex-seq CAGI 5 training set TR957, verifying that ΔMES5 is the single most correlated feature (PCC = 0.402) and that nine features exceed |PCC| > 0.1 as in Table 1 of the PredPSI-SVR paper.

## When to use

Use when you have the vex-seq challenge training set TR957 with extracted splicing features and experimental ΔΨ values, and need to rank individual features by correlation with ΔΨ or validate the Table 1 feature ranking (ΔMES5, PCC = 0.402).

## When NOT to use

- The goal is to predict ΔΨ for new variants with the final PredPSI-SVR model rather than to analyze feature correlations on the training set
- You need greedy feature selection to derive the final eight-feature model, not the per-feature correlation ranking of all 42 features
- The data are from the vex-seq blind test set or another cohort rather than the TR957 training set with experimental ΔΨ

## Inputs

- vex-seq challenge training dataset TR957 with experimental ΔΨ values (CAGI 5)
- Matrix of 42 comprehensive splicing-related features extracted from each exon sequence and its flanking regions
- PredPSI-SVR repository (feature extraction pipeline: Perl, Python 2/3 with numpy, ANNOVAR ensGene hg19 database, SPIDEX, hg19 genome)

## Outputs

- feature_pcc_ranking.csv with per-feature PCC values and ranks for all 42 features
- Validated list of nine features with |PCC| > 0.1 matching Table 1
- Top-ranked feature ΔMES5 with PCC = 0.402

## How to apply

Load the TR957 training set (the vex-seq challenge training dataset from the 5th Critical Assessment of Genome Interpretation) containing the 42 comprehensive splicing-related features extracted from each exon and its flanking regions plus experimental ΔΨ values. For each of the 42 features, compute the Pearson correlation coefficient against experimental ΔΨ using Python 3 with the numpy package (PredPSI-SVR also requires Perl in PATH for its feature-extraction pipeline). Rank the features by absolute PCC and select the subset with |PCC| > 0.1. Validate against Table 1: the computed PCC of ΔMES5 must equal the reported 0.402, ΔMES5 must be top-ranked, and exactly nine features must exceed the |PCC| = 0.1 threshold. Write the per-feature PCC values and ranks to feature_pcc_ranking.csv. The article notes that features on splicing sites computed by MaxEntScan have the highest correlations, so splice-site features are expected near the top of the ranking.

## Related tools

- **PredPSI-SVR** (Source of the TR957 training dataset pipeline and the 42 extracted splicing features whose PCC against ΔΨ is computed) — https://github.com/chenkenbio/PredPSI-SVR
- **numpy (Python 3)** (Computes the Pearson correlation coefficient of each of the 42 features against experimental ΔΨ)
- **Perl** (Runtime dependency of the PredPSI-SVR feature-extraction pipeline (e.g. MaxEntScan-based splice-site scoring) that must be in PATH)

## Examples

```
git clone https://github.com/chenkenbio/PredPSI-SVR && python3 -c "import numpy as np; r = np.corrcoef(X['dMES5'], y_dpsi)[0,1]"
```

## Evaluation signals

- Computed PCC of ΔMES5 equals the reported value 0.402 and ΔMES5 is the top-ranked feature
- Exactly nine features have |PCC| > 0.1 on TR957 and the set matches Table 1 of the paper
- All 42 splicing-related features are present in feature_pcc_ranking.csv, each with a finite PCC and a rank
- Correlations are computed against experimental ΔΨ on the training set TR957, not on the blind test set or filtered subsets
- Splice-site features (e.g. MaxEntScan-based scores) rank near the top, consistent with the article's discussion

## Limitations

- The training set is small; removing ΔΨ outliers with absolute values less than two times the standard deviation was needed to improve correlations for all methods (53 mutations remained), so PCCs are sensitive to sample filtering
- PredPSI-SVR does not include branch point sequence features or pre-mRNA secondary structure features effectively due to the limited number of samples, so the 42-feature space omits these signals
- Without experimental wild-type Ψ, performance drops by about 0.1 PCC, so downstream modeling on top of this ranking requires wild-type Ψ input
- Many exons cannot be analyzed via MISO due to sequencing depth, since only Human BodyMap Project 2.0 RNA-seq data were used

## Evidence
