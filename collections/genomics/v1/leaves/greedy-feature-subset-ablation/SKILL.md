---
name: greedy-feature-subset-ablation
description: Use when a greedy feature selection has produced a fixed feature subset
  for an SVR model and you want to verify by ablation that removing specific features
  (e.g. conservation or distance-based ones) improves CV and blind-test PCC.
when_to_use_negative:
- You need the published challenge performance (PCC 0.570 CV / 0.566 blind) of the
  original 8-feature model rather than the ablation study
- Your variants are in intergenic regions or more than 200 bp from splice sites —
  PredPSI-SVR filters these out before feature extraction, so delta-PSI prediction
  is not applicable
- The dataset is large enough that deep-learning methods (e.g. MMSplice-style neural
  networks) are preferable; the article notes the small vex-seq sample size prevented
  deep learning
tools:
- name: PredPSI-SVR
  role: Source pipeline for extracting the 42 splicing-related features and the 8-feature
    greedy-selected SVR baseline; the 6-feature ablated server model is derived from
    it
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression implementation used to train and cross-validate
    the ablated 6-feature model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:9f5bb4406017f150943527d1a45ae073e6aeb23d3f8dbdfdc7c7c3e1bdb654eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - PredPSI-SVR
  - libsvm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# greedy-feature-subset-ablation

## Summary

Tests whether a greedy feature-selection result is globally optimal by ablating selected features from a support vector regression model (here, dropping the CADD-derived priPhyloP and minDistTSS from PredPSI-SVR's 8-feature set) and re-evaluating on 10-fold cross-validation and an independent blind test.

## When to use

Use when a greedy feature selection has produced a fixed feature subset for an SVR model and you want to verify by ablation that removing specific features (e.g. conservation or distance-based ones) improves CV and blind-test PCC.

## When NOT to use

- You need the published challenge performance (PCC 0.570 CV / 0.566 blind) of the original 8-feature model rather than the ablation study
- Your variants are in intergenic regions or more than 200 bp from splice sites — PredPSI-SVR filters these out before feature extraction, so delta-PSI prediction is not applicable
- The dataset is large enough that deep-learning methods (e.g. MMSplice-style neural networks) are preferable; the article notes the small vex-seq sample size prevented deep learning

## Inputs

- Vex-seq challenge training dataset (CAGI 5, TR957 + TS1098) with measured delta-PSI values
- 42 splicing-related features extracted from exon and flanking sequences (incl. priPhyloP and minDistTSS)
- Independent blind test set (TS1098) with measured delta-PSI values

## Outputs

- pcc_results.csv containing 10-fold CV PCC and independent-test PCC for the ablated 6-feature SVR model
- Retrained 6-feature libsvm SVR model (final server version)

## How to apply

Start from the greedy-selected 8-feature PredPSI-SVR model trained with libsvm on the combined TR957+TS1098 vex-seq training data. Extract the 42 splicing-related features, then build a 6-feature subset by removing priPhyloP (conservation) and minDistTSS (distance to transcription start site). Retrain the support vector regression model with libsvm on the combined TR957+TS1098 vex-seq training data using only the 6-feature subset. Evaluate by 10-fold cross-validation on the combined training set and on the independent test set (TS1098), computing the Pearson correlation coefficient (PCC) between predicted and measured delta-PSI values, and write both PCC values to pcc_results.csv. The ablation is judged successful if the reproduced PCCs match the reported 0.590 (10-fold CV) and 0.577 (independent test) within a reasonable tolerance, demonstrating that the greedy algorithm's 8-feature choice is not globally optimal.

## Related tools

- **PredPSI-SVR** (Source pipeline for extracting the 42 splicing-related features and the 8-feature greedy-selected SVR baseline; the 6-feature ablated server model is derived from it) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression implementation used to train and cross-validate the ablated 6-feature model)

## Examples

```
./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Reproduced 10-fold CV PCC should match 0.590 within a reasonable tolerance
- Reproduced independent (blind) test PCC should match 0.577 within a reasonable tolerance
- Both ablated-model PCCs must exceed the 8-feature model's CV PCC of 0.570 and blind PCC of 0.566 to support the ablation conclusion
- pcc_results.csv must contain exactly two PCC values (10-fold CV and independent test) keyed to their respective evaluation sets
- Feature matrix for the ablated model must contain exactly 6 columns with priPhyloP and minDistTSS absent

## Limitations

- Requires experimental wild-type PSI of exons as input; without it, PCCs on training and test sets drop by about 0.1
- Small number of vex-seq samples prevented use of more powerful algorithms like deep learning and limited inclusion of branch point and pre-mRNA secondary structure features
- Only Human BodyMap Project 2.0 RNA-seq data used; many exons cannot be recovered due to sequencing depth, so the model is not tissue-specific

## Evidence
