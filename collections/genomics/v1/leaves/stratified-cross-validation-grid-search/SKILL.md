---
name: stratified-cross-validation-grid-search
description: Use when assembling a supervised quality-score model for structural variant
  calls from labelled positive/negative feature vectors (e.g. GIAB HG001/HG002, svclassify,
  NanoVar gold sets) and you need feature selection plus hyperparameter tuning before
  serializing the model.
when_to_use_negative:
- You only need to score/filter existing calls with a trained model — use dysgu filter
  with model-probability thresholds (≥0.5 for PacBio, ≥0.35 for ONT) instead of retraining
- Only simulated training data are available — the dysgu authors specifically avoided
  simulated datasets, considering non-simulated training to enhance performance
- The input is already a fitted model pickle; re-running feature elimination and grid
  search is unnecessary for inference
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: dysgu
  role: Supplies the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df,
    dysgu_default_args/dysgu_preset_args) for labelling gold sets, computing SV feature
    vectors, and serializing the fitted model to dysgu_model.pickle
  repo: https://github.com/kcleal/dysgu
- name: lightgbm
  role: Gradient-boosting 'dart' classifier backend used to label putative SV events
    with quality-score probabilities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:75c613c5f06fb92822d7898dde1543c1fbfee09d9e4224da66fc89b8829c1bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - lightgbm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# stratified-cross-validation-grid-search

## Summary

Trains dysgu's SV quality-score classifier by combining recursive feature elimination with a 5-fold stratified cross-validated grid search over hyperparameters of a LightGBM 'dart' gradient-boosting model, using non-simulated gold-set labelled SVs as training data.

## When to use

Use when assembling a supervised quality-score model for structural variant calls from labelled positive/negative feature vectors (e.g. GIAB HG001/HG002, svclassify, NanoVar gold sets) and you need feature selection plus hyperparameter tuning before serializing the model.

## When NOT to use

- You only need to score/filter existing calls with a trained model — use dysgu filter with model-probability thresholds (≥0.5 for PacBio, ≥0.35 for ONT) instead of retraining
- Only simulated training data are available — the dysgu authors specifically avoided simulated datasets, considering non-simulated training to enhance performance
- The input is already a fitted model pickle; re-running feature elimination and grid search is unnecessary for inference

## Inputs

- pbsv HG005 PacBio deletion/insertion VCF (GIAB FTP)
- HG001 svclassify/Personalis deletion call set BED (deletions ≥50 bp)
- NanoVar-derived insertion call set
- Putative SV feature vectors computed with the DysguSV API (dysgu_default_args / dysgu_preset_args)

## Outputs

- dysgu_model.pickle (serialized LightGBM 'dart' classifier)
- feature_importance.csv (sorted feature-importance ranking, FCC second only to SV length)
- Selected feature subset and tuned hyperparameters

## How to apply

Build labelled training data from non-simulated gold sets: load the pbsv HG005 PacBio deletion/insertion VCF, the HG001 svclassify/Personalis deletion call set (deletions ≥50 bp), and the NanoVar insertion call set via dysgu's load_dysgu_vcf and load_bed API functions, then reconcile overlaps with merge_intervals and merge_dysgu_df to assign positive labels. Compute per-event SV feature vectors with the DysguSV API using parameters from dysgu_default_args / dysgu_preset_args. Train a LightGBM 'dart' gradient-boosting classifier, then perform recursive feature elimination combined with 5-fold stratified cross-validated grid search to select the final feature subset and hyperparameters. The rationale is that training on non-simulated (rather than simulated, as nanovar does with its 14 neural-network input features) datasets improves classification performance. Serialize the fitted model to dysgu_model.pickle and export the sorted feature-importance ranking to feature_importance.csv; the fold change in coverage (FCC) metric should rank immediately after SV length.

## Related tools

- **dysgu** (Supplies the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df, dysgu_default_args/dysgu_preset_args) for labelling gold sets, computing SV feature vectors, and serializing the fitted model to dysgu_model.pickle) — https://github.com/kcleal/dysgu
- **lightgbm** (Gradient-boosting 'dart' classifier backend used to label putative SV events with quality-score probabilities)

## Examples

```
python -c "from dysgu.io import load_dysgu_vcf, load_bed, merge_intervals; df = load_dysgu_vcf('HG005_pbsv.vcf'); bed = load_bed('HG001_deletions.bed'); pos = merge_intervals(bed)"
```

## Evaluation signals

- feature_importance.csv places FCC immediately after SV length in the sorted ranking
- dysgu_model.pickle loads successfully via the DysguSV API
- A script_runs check on the training script passes
- 5-fold stratified cross-validation yields stable hyperparameters across folds (no fold-dependent feature-subset collapse)
- Downstream filtering with the trained model uses max model probability thresholds ≥0.5 (PacBio) / ≥0.35 (ONT) as in the dysgu merge workflow

## Limitations

- Manually labelling the training set induces an obvious bias, where correctness is a matter of opinion of the human observer
- The feature list is non-exhaustive and may be expanded in future dysgu releases, so the trained feature subset may not match later software versions
- Dysgu's heuristics (span position distance, minimizer support) are unoptimized and could be tuned with a larger test set, limiting training-set quality

## Evidence

- [discussion] Putative events are then labelled with a useful probability value using a gradient boosting classifier (30).: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
- [intro] we considered that performance may be enhanced from training using non-simulated datasets: "we considered that performance may be enhanced from training using non-simulated datasets"
- [supplementary] Data used to train the classifier is available online at https://zenodo.org/record/4761527.: "Data used to train the classifier is available online at https://zenodo.org/record/4761527."
- [discussion] Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer.: "Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer."
- [results] filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations"
