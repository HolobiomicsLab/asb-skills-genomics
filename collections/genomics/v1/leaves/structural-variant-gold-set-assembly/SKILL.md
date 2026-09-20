---
name: structural-variant-gold-set-assembly
description: Use when a machine-learning SV caller needs labelled positive/negative
  training events, i.e. when real (non-simulated) gold-set call sets must be downloaded,
  interval-merged and converted into labelled feature vectors.
when_to_use_negative:
- The caller already ships a trained model and you only need to score/filter calls
  (e.g. filter on model probability ≥0.5 for PacBio or ≥0.35 for ONT) rather than
  rebuild the training set
- The input is a single-sample benchmark evaluation task (comparing query VCFs against
  a truth set with svbench) rather than training-set construction
- Only simulated-data training is intended and no real gold-set call sets are required
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the python-API functions (load_dysgu_vcf, load_bed, merge_intervals,
    merge_dysgu_df) and DysguSV API used to load gold sets, merge intervals, and compute
    per-event SV feature vectors
  repo: https://github.com/kcleal/dysgu
- name: nanovar
  role: Source of the long-read insertion call set used as a gold-set training resource
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:a7afb0570662360e2a17e18b2ab539bae66b7e5c035c4712e82977c8f8d029dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - nanovar
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# structural-variant-gold-set-assembly

## Summary

Assembles labelled structural-variant training (gold) sets by downloading public benchmark call sets — e.g. GIAB pbsv HG005 PacBio deletion/insertion VCF, HG001 svclassify/Personalis deletion BED (≥50 bp), and NanoVar-derived insertions — reconciling overlaps and assigning positive labels for classifier training.

## When to use

Use when a machine-learning SV caller needs labelled positive/negative training events, i.e. when real (non-simulated) gold-set call sets must be downloaded, interval-merged and converted into labelled feature vectors.

## When NOT to use

- The caller already ships a trained model and you only need to score/filter calls (e.g. filter on model probability ≥0.5 for PacBio or ≥0.35 for ONT) rather than rebuild the training set
- The input is a single-sample benchmark evaluation task (comparing query VCFs against a truth set with svbench) rather than training-set construction
- Only simulated-data training is intended and no real gold-set call sets are required

## Inputs

- pbsv HG005 PacBio deletion/insertion VCF from the GIAB FTP site
- HG001 svclassify/Personalis deletion call set (deletions ≥50 bp) as a BED file
- NanoVar-derived insertion call set
- Putative SV call set / feature vectors to be labelled (dysgu call output)

## Outputs

- Merged, non-overlapping gold-set intervals with positive labels (via merge_intervals / merge_dysgu_df)
- Labelled per-event SV feature vectors (including SV length and FCC)
- Trained LightGBM model serialized as dysgu_model.pickle
- Sorted feature-importance ranking exported as feature_importance.csv

## How to apply

Download the gold-set resources (GIAB FTP pbsv HG005 PacBio deletion/insertion VCF; HG001 svclassify/Personalis deletion call set ≥50 bp as BED; NanoVar-derived insertion call set) and load them with dysgu's python API functions load_dysgu_vcf (for VCF) and load_bed (for BED). Reconcile overlapping gold regions using merge_intervals / merge_dysgu_df so reciprocal overlapping calls collapse into single positive-label intervals; the article notes non-simulated training data was chosen deliberately because 'performance may be enhanced from training using non-simulated datasets', avoiding the bias of simulated sets. For each putative event compute the SV feature vector (including SV length and the fold change in coverage, FCC, metric) using the DysguSV API with parameters from dysgu_default_args / dysgu_preset_args. The labelled intervals then feed a LightGBM 'dart' gradient-boosting classifier, tuned by recursive feature elimination with 5-fold stratified cross-validated grid search, yielding dysgu_model.pickle and feature_importance.csv.

## Related tools

- **dysgu** (Provides the python-API functions (load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df) and DysguSV API used to load gold sets, merge intervals, and compute per-event SV feature vectors) — https://github.com/kcleal/dysgu
- **nanovar** (Source of the long-read insertion call set used as a gold-set training resource)

## Examples

```
from dysgu.io import load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df; gold = merge_intervals(load_dysgu_vcf('HG005_pbsv.vcf'), load_bed('HG001_svclassify_dels.bed'))
```

## Evaluation signals

- Merged gold intervals contain no reciprocal overlaps after merge_intervals / merge_dysgu_df reconciliation
- Exported feature_importance.csv ranks FCC immediately after SV length
- dysgu_model.pickle reloads successfully via the DysguSV API
- A script_runs check on the training script passes
- Gold-set labels trace back to the expected sources: GIAB pbsv HG005 VCF, HG001 svclassify deletions ≥50 bp, and NanoVar insertions

## Limitations

- Manually labelling training events induces observer bias, where correctness is a matter of opinion of the human observer
- The classifier feature list is non-exhaustive and may be expanded in future releases, so trained models may not transfer across dysgu versions
- Training-data hosting/accession details are not fully given in the paper; the classifier training data is hosted at https://zenodo.org/record/4761527

## Evidence

- [intro] we considered that performance may be enhanced from training using non-simulated datasets: "we considered that performance may be enhanced from training using non-simulated datasets"
- [discussion] Putative events are then labelled with a useful probability value using a gradient boosting classifier: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
- [results] we assessed recall on the HG001 cell line that has corresponding deletion calls (≥50 bp) provided by GIAB: "we assessed recall on the HG001 cell line that has corresponding deletion calls (≥50 bp) provided by GIAB"
- [supplementary] Data used to train the classifier is available online at zenodo record 4761527: "Data used to train the classifier is available online at https://zenodo.org/record/4761527."
- [intro] The nanovar LR caller uses a neural network classifier trained on simulated datasets with 14 input features: "The LR caller nanovar utilizes a neural network classifier trained on simulated datasets, where 14 input features of each putative SV are used to classify events (17)"
