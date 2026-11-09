---
name: feature-importance-ranking-interpretation
description: Use when a trained SV-calling classifier (e.g. dysgu's model) needs its
  feature-importance ranking examined to identify which per-event metrics—after SV
  length—most influence quality-score predictions.
when_to_use_negative:
- The input is already a feature-importance table or ranking and only needs reading,
  not retraining or re-ranking
- The goal is SV calling/benchmarking performance (precision/recall on GIAB HG002
  or syndip) rather than interpreting the quality-classification model
- Training data are simulated rather than the non-simulated gold-set call sets, since
  the article notes performance is enhanced by training on non-simulated datasets
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: dysgu
  role: Provides the LightGBM gradient-boosting classifier, per-event SV feature computation
    (DysguSV API, dysgu_default_args/dysgu_preset_args), and I/O helpers (load_dysgu_vcf,
    load_bed, merge_intervals, merge_dysgu_df) used to train and export the feature-importance
    ranking
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:59631a324f04f5f116f5f2f596736f956268dd818fa470366f2131f0b354f206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# feature-importance-ranking-interpretation

## Summary

Interprets the ranked feature-importance output of dysgu's LightGBM gradient-boosting SV quality classifier, which places the fold change in coverage (FCC) metric immediately after SV length, to understand which per-event signals drive variant quality scores.

## When to use

Use when a trained SV-calling classifier (e.g. dysgu's model) needs its feature-importance ranking examined to identify which per-event metrics—after SV length—most influence quality-score predictions.

## When NOT to use

- The input is already a feature-importance table or ranking and only needs reading, not retraining or re-ranking
- The goal is SV calling/benchmarking performance (precision/recall on GIAB HG002 or syndip) rather than interpreting the quality-classification model
- Training data are simulated rather than the non-simulated gold-set call sets, since the article notes performance is enhanced by training on non-simulated datasets

## Inputs

- Labelled SV feature vectors (per-event metrics such as SV length, FCC, read support) for putative variants
- Gold-set call sets: pbsv HG005 PacBio deletion/insertion VCF, HG001 svclassify/Personalis deletion BED (≥50 bp), NanoVar insertion call set
- Fitted LightGBM classifier (dysgu_model.pickle)

## Outputs

- feature_importance.csv with sorted feature-importance ranking (FCC expected immediately after SV length)
- Serialized fitted model dysgu_model.pickle
- Selected feature subset and hyperparameters from RFE + 5-fold stratified CV grid search

## How to apply

Train the LightGBM 'dart' gradient-boosting classifier on labelled SV feature vectors assembled from three gold sets (pbsv HG005 PacBio deletion/insertion VCF from GIAB FTP, the HG001 svclassify/Personalis deletion call set as BED with deletions ≥50 bp, and the NanoVar-derived insertion call set), labelling positives via merge_intervals/merge_dysgu_df reconciliation. Optimize the feature subset with recursive feature elimination combined with 5-fold stratified cross-validated grid search, then export the sorted feature-importance ranking to feature_importance.csv alongside the serialized dysgu_model.pickle. Interpret the ranking by checking that FCC (fold change in coverage, a read-depth metric around break sites) is positioned immediately after SV length, confirming that coverage-change signals—not just event size—drive the model probability used to prioritize variants (thresholded downstream at ≥0.5 for PacBio and ≥0.35 for ONT).

## Related tools

- **dysgu** (Provides the LightGBM gradient-boosting classifier, per-event SV feature computation (DysguSV API, dysgu_default_args/dysgu_preset_args), and I/O helpers (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df) used to train and export the feature-importance ranking) — https://github.com/kcleal/dysgu

## Examples

```
import lightgbm as lgb; model = lgb.train(params, lgb.Dataset(X, y)); pd.DataFrame({'feature': X.columns, 'gain': model.feature_importance('gain')}).sort_values('gain', ascending=False).to_csv('feature_importance.csv', index=False)
```

## Evaluation signals

- dysgu_model.pickle loads successfully via the DysguSV API
- The exported ranking in feature_importance.csv places FCC immediately after SV length
- A script_runs check on the training script passes
- Hyperparameters and feature subset reproduce the RFE + 5-fold stratified CV grid-search selection
- Downstream filtering thresholds (max model probability ≥0.5 PacBio, ≥0.35 ONT) remain consistent with the trained model's probability scale

## Limitations

- The feature list is non-exhaustive and may expand in future dysgu releases, changing the ranking
- Manually labelled training sets induce observer bias, since correctness is a matter of opinion of the human observer
- Dysgu's heuristics (span position distance, minimizer support) are unoptimized and could be tuned with a larger test set

## Evidence

- [intro] A machine learning classifier is then employed to generate a useful quality score which can be used to prioritize variants.: "A machine learning classifier is then employed to generate a useful quality score which can be used to prioritize variants."
- [supplementary] Training data available on Zenodo: "Data used to train the classifier is available online at https://zenodo.org/record/4761527."
- [results] Downstream probability filtering thresholds: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations"
- [discussion] Observer bias in labelled training sets: "Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer."
