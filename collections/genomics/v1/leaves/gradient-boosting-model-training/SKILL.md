---
name: gradient-boosting-model-training
description: Use when training or retraining dysgu's SV quality-classification model
  from real (non-simulated) benchmark call sets, or when reproducing/validating the
  feature-importance ranking of dysgu's per-event SV features.
when_to_use_negative:
- The model already exists and you only want to apply it — dysgu supports switching
  models post-calling via the python-API without retraining
- You only need to prioritize/filter dysgu calls — the fitted classifier already outputs
  probability values filterable at ≥0.5 (PacBio) or ≥0.35 (ONT), so no retraining
  is required
- Your inputs are already labelled per-event feature tables awaiting model inference
  rather than raw call sets needing label reconciliation
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df,
    DysguSV, dysgu_default_args/dysgu_preset_args) used to assemble labelled training
    data and host the gradient-boosting quality-score classifier
  repo: https://github.com/kcleal/dysgu
- name: nanovar
  role: 'Contrast case: nanovar''s classifier is trained on simulated datasets, whereas
    this skill trains dysgu''s classifier on non-simulated gold-set data'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:4eada3d279374310f188820f7d60312179dd34a472b0ce8e06dd54817087a424
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - nanovar
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gradient-boosting-model-training

## Summary

Trains dysgu's gradient-boosting (LightGBM 'dart') quality-score classifier for structural variant events from non-simulated gold-set call sets (GIAB HG005 pbsv PacBio del/ins VCF, HG001 svclassify/Personalis deletion BED, NanoVar insertion set), then serializes the fitted model and feature-importance ranking.

## When to use

Use when training or retraining dysgu's SV quality-classification model from real (non-simulated) benchmark call sets, or when reproducing/validating the feature-importance ranking of dysgu's per-event SV features.

## When NOT to use

- The model already exists and you only want to apply it — dysgu supports switching models post-calling via the python-API without retraining
- You only need to prioritize/filter dysgu calls — the fitted classifier already outputs probability values filterable at ≥0.5 (PacBio) or ≥0.35 (ONT), so no retraining is required
- Your inputs are already labelled per-event feature tables awaiting model inference rather than raw call sets needing label reconciliation

## Inputs

- pbsv HG005 PacBio deletion/insertion VCF from the GIAB FTP site (gold-set positives)
- HG001 svclassify/Personalis deletion call set as BED file (deletions ≥50 bp)
- NanoVar-derived insertion call set
- Putative SV feature vectors computed via the DysguSV API (dysgu_default_args / dysgu_preset_args parameters)
- Classifier training data archive from https://zenodo.org/record/4761527

## Outputs

- dysgu_model.pickle — serialized fitted LightGBM 'dart' gradient-boosting classifier
- feature_importance.csv — sorted feature-importance ranking (FCC immediately after SV length)
- Labelled SV feature vectors with positive/negative quality labels

## How to apply

Download the three gold-set resources (the pbsv HG005 PacBio deletion/insertion VCF from the GIAB FTP site, the HG001 svclassify/Personalis deletion call set (deletions ≥50 bp) as BED, and the NanoVar-derived insertion call set), then load them with the dysgu python-API functions load_dysgu_vcf and load_bed, reconciling overlapping gold regions with merge_intervals / merge_dysgu_df to assign positive labels. Compute per-event SV feature vectors with the DysguSV API using parameters from dysgu_default_args / dysgu_preset_args, then train a LightGBM 'dart' gradient-boosting classifier on the labelled vectors. Select the final feature subset and hyperparameters via recursive feature elimination combined with 5-fold stratified cross-validated grid search; serialize the fitted model to dysgu_model.pickle and export the sorted feature-importance ranking to feature_importance.csv. The rationale is that dysgu's authors deliberately trained on non-simulated datasets (unlike, e.g., nanovar's neural network trained on simulated data), and training data are hosted at zenodo record 4761527.

## Related tools

- **dysgu** (Provides the python-API (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df, DysguSV, dysgu_default_args/dysgu_preset_args) used to assemble labelled training data and host the gradient-boosting quality-score classifier) — https://github.com/kcleal/dysgu
- **nanovar** (Contrast case: nanovar's classifier is trained on simulated datasets, whereas this skill trains dysgu's classifier on non-simulated gold-set data)

## Evaluation signals

- dysgu_model.pickle loads successfully via the DysguSV API
- feature_importance.csv places the fold-change-in-coverage (FCC) metric immediately after SV length in the sorted ranking
- A script_runs check on the training script passes
- The applied model's probability outputs support downstream filtering at max model probability ≥0.5 for PacBio and ≥0.35 for ONT combinations
- Label reconciliation with merge_intervals / merge_dysgu_df assigns positives only to events overlapping the gold-set regions

## Limitations

- Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer
- The feature list is non-exhaustive and may expand in future dysgu releases, so retrained models may not match the published ranking
- Manual post-hoc filtering is replaced by probability stratification, so thresholds (≥0.5 PacBio, ≥0.35 ONT) must be re-tuned if the model is retrained on different data

## Evidence

- [discussion] Putative events are then labelled with a useful probability value using a gradient boosting classifier (30).: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
- [intro] we considered that performance may be enhanced from training using non-simulated datasets: "we considered that performance may be enhanced from training using non-simulated datasets"
- [supplementary] Data used to train the classifier is available online at https://zenodo.org/record/4761527.: "Data used to train the classifier is available online at https://zenodo.org/record/4761527."
- [discussion] Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer.: "Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer."
- [intro] The LR caller nanovar utilizes a neural network classifier trained on simulated datasets, where 14 input features of each putative SV are used to classify events: "The LR caller nanovar utilizes a neural network classifier trained on simulated datasets, where 14 input features of each putative SV are used to classify events (17)"
