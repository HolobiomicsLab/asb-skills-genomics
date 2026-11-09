---
name: recursive-feature-elimination
description: Use when training a machine-learning quality-score classifier for structural
  variant calls from gold-set (non-simulated) training data and needing a feature-importance
  ranking to prune the feature vector.
when_to_use_negative:
- Input data is a finished VCF needing filtering only — apply dysgu's filter command
  with the model probability thresholds (≥0.5 PacBio, ≥0.35 ONT) instead of retraining.
- Training data is simulated rather than from gold-set call sets — the paper found
  performance may be enhanced by training on non-simulated datasets, so simulated-data
  training is the weaker choice.
- The goal is calling SVs from reads, not building or evaluating the quality-score
  model.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df,
    DysguSV, dysgu_default_args, dysgu_preset_args) used to build labelled feature
    vectors and train/serve the quality-score model
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:80bc736f452fd61c8982890ee0d2a308cf1d87b27e61b7cbb84b79703a05af13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# recursive-feature-elimination

## Summary

Assemble dysgu's SV quality-score training data from non-simulated gold-set call sets and train a LightGBM 'dart' gradient-boosting classifier, using recursive feature elimination with 5-fold stratified cross-validated grid search to select the final feature subset and hyperparameters.

## When to use

Use when training a machine-learning quality-score classifier for structural variant calls from gold-set (non-simulated) training data and needing a feature-importance ranking to prune the feature vector.

## When NOT to use

- Input data is a finished VCF needing filtering only — apply dysgu's filter command with the model probability thresholds (≥0.5 PacBio, ≥0.35 ONT) instead of retraining.
- Training data is simulated rather than from gold-set call sets — the paper found performance may be enhanced by training on non-simulated datasets, so simulated-data training is the weaker choice.
- The goal is calling SVs from reads, not building or evaluating the quality-score model.

## Inputs

- pbsv HG005 PacBio deletion/insertion VCF from the GIAB FTP site
- HG001 svclassify/Personalis deletion call set (deletions ≥50 bp) as a BED file
- NanoVar-derived insertion call set
- Putative SV events with per-event feature vectors computed via the DysguSV API

## Outputs

- Serialized classifier model (dysgu_model.pickle)
- Sorted feature-importance ranking (feature_importance.csv), with FCC ranked immediately after SV length
- Trained LightGBM 'dart' classifier producing probability values for putative SV events

## How to apply

Download the gold-set resources: the pbsv HG005 PacBio deletion/insertion VCF from GIAB FTP, the HG001 svclassify/Personalis deletion call set (deletions ≥50 bp) as BED, and the NanoVar-derived insertion call set; classifier training data is also hosted at zenodo.org/record/4761527. Load these with dysgu's python-API functions load_dysgu_vcf and load_bed, then reconcile overlapping gold regions using merge_intervals / merge_dysgu_df to assign positive labels. Compute per-event SV feature vectors for every putative variant via the DysguSV API using parameters from dysgu_default_args / dysgu_preset_args, then train a LightGBM 'dart' gradient-boosting classifier on the labelled vectors. Select the final feature subset and hyperparameters by combining recursive feature elimination with a 5-fold stratified cross-validated grid search. Serialize the fitted model to dysgu_model.pickle and export the sorted feature-importance ranking to feature_importance.csv; the paper reports the fold change in coverage (FCC) metric as the most important feature after SV length.

## Related tools

- **dysgu** (Provides the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df, DysguSV, dysgu_default_args, dysgu_preset_args) used to build labelled feature vectors and train/serve the quality-score model) — https://github.com/kcleal/dysgu

## Examples

```
python train_model.py --gold-vcf HG005_pbsv.vcf --gold-bed HG001_del.bed --model-out dysgu_model.pickle --importance-out feature_importance.csv
```

## Evaluation signals

- dysgu_model.pickle loads successfully via the DysguSV API
- feature_importance.csv places the fold change in coverage (FCC) metric immediately after SV length
- A script_runs check on the training script passes
- Model probability values discriminate true events: filtering calls with maximum model probability ≥0.5 for PacBio or ≥0.35 for ONT combinations
- Labelled positives derive from reconciled (merged) overlaps across the three gold-set call sets, not raw unmerged intervals

## Limitations

- Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer
- The current list of features is non-exhaustive and can potentially be expanded in future releases to enhance calling performance
- No complete feature enumeration or dysgu version is stated in the article

## Evidence

- [other] Train a LightGBM 'dart' gradient-boosting classifier on the labelled feature vectors: [evidence span withheld — anchor status: not_found]
- [intro] Non-simulated training rationale: "we considered that performance may be enhanced from training using non-simulated datasets"
- [discussion] Gradient boosting labels putative events with probability: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
- [supplementary] Training data hosted on Zenodo: "Data used to train the classifier is available online at https://zenodo.org/record/4761527."
- [results] Probability thresholds for filtering: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations"
- [discussion] Manual labelling bias caveat: "Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer."
