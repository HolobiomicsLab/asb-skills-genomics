---
name: nanopore-feature-ablation
description: Use when reproducing or extending ablation experiments on Nanoident's
  multi-column CNN to determine which nanopore feature or architectural component
  drives methylation typing and fine-mapping accuracy.
when_to_use_negative:
- The goal is de novo methylation motif discovery or enrichment rather than quantifying
  component contributions to Nanoident accuracy
- Input data are raw nanopore signals not yet preprocessed into R/pkl feature tables
  via the nanodisco/update.sh and data_from_R_to_python.ipynb pipeline
- Working with modkit/bedMethyl pileup modification calls rather than the Nanoident
  per-motif feature representation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: Nanoident
  role: Multi-column CNN whose features, loss function, and modules are individually
    ablated and retrained
  repo: https://github.com/cz-csu/Nanoident
- name: conda
  role: Environment setup via Nanoident/env/freeze.yml for Python model training
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Upstream preprocessing that originally handled only the mean feature; update.sh
    extends it to the four additional Nanoident features
  repo: https://github.com/fanglab/nanodisco
- name: Python
  role: Runs the ablation training/evaluation scripts (train_loocv_model.py, train_loocv_test.py,
    train_test.py)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:871d9086d95563090a3a1da4ab2f118cc00f114b868f4f5edca976b15c1bc372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Python
  - conda
  - nanodisco
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# nanopore-feature-ablation

## Summary

Reproduces Nanoident's feature- and module-ablation study: individually removing each of the five input features (mean, std, time, proportion of reference bases, average quality score), swapping Focal Loss for Cross-Entropy Loss, and dropping residual connections or the weight block, to quantify each component's contribution to methylation typing/fine-mapping accuracy.

## When to use

Use when reproducing or extending ablation experiments on Nanoident's multi-column CNN to determine which nanopore feature or architectural component drives methylation typing and fine-mapping accuracy.

## When NOT to use

- The goal is de novo methylation motif discovery or enrichment rather than quantifying component contributions to Nanoident accuracy
- Input data are raw nanopore signals not yet preprocessed into R/pkl feature tables via the nanodisco/update.sh and data_from_R_to_python.ipynb pipeline
- Working with modkit/bedMethyl pileup modification calls rather than the Nanoident per-motif feature representation

## Inputs

- Converted pkl feature files with Nanoident's five features (mean, std, time, proportion of reference bases, average quality score) plus the basecalling feature
- 7-bacteria / 46-motif training dataset (LOOCV setting)
- Independent test dataset with 12 methylation motifs
- Nanoident model training scripts (train_loocv_model.py, train_loocv_test.py, train_test.py)

## Outputs

- Ablation accuracy table (e.g. ablation_results.csv) with columns for the removed component and the evaluation setting (LOOCV vs independent test set)
- Per-feature accuracy deltas relative to the full Nanoident model
- Loss-function comparison (Focal Loss vs Cross-Entropy Loss) and architecture-variant accuracies (residual-connection removal, weight-block removal)

## How to apply

Load the converted pkl feature files containing the five Nanoident features plus the basecalling feature, then for each ablation (time, bnf, ab, basecalling, std) drop that feature column and retrain the multi-column CNN with train_loocv_model.py, evaluating with train_loocv_test.py under leave-one-out cross-validation on the 7-bacteria/46-motif dataset. Repeat each feature ablation on the independent test set using train_test.py. Train loss-function variants replacing Focal Loss with Cross-Entropy Loss under both LOOCV and the independent set, and retrain variants with residual connections and the weight block removed. Record the accuracy delta of each ablated model relative to the full model and aggregate into a results table (e.g. ablation_results.csv).

## Related tools

- **Nanoident** (Multi-column CNN whose features, loss function, and modules are individually ablated and retrained) — https://github.com/cz-csu/Nanoident
- **conda** (Environment setup via Nanoident/env/freeze.yml for Python model training) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Upstream preprocessing that originally handled only the mean feature; update.sh extends it to the four additional Nanoident features) — https://github.com/fanglab/nanodisco
- **Python** (Runs the ablation training/evaluation scripts (train_loocv_model.py, train_loocv_test.py, train_test.py))

## Examples

```
python ./model/train_loocv_model.py  # retrain multi-column CNN with one feature column dropped; then python ./model/train_loocv_test.py to evaluate under LOOCV
```

## Evaluation signals

- Focal Loss should outperform Cross-Entropy Loss by ~1% under both LOOCV and the independent test set
- Removing residual connections should cost ~1.1% accuracy and removing the weight block ~0.25% relative to the full model
- The mean feature should show the largest accuracy drop when removed, followed by std; time, bnf, and ab should contribute relatively little
- Delta directions and magnitudes should be consistent between the 7-bacteria/46-motif LOOCV setting and the independent 12-motif test set

## Limitations

- Ablations must be run separately under LOOCV and the independent test set; results from one setting do not automatically transfer
- Preprocessing is memory-constrained — nanodisco preprocessing must be done in chunks and then merged, so regenerating ablated feature sets can be slow
- The original nanodisco container only handles the mean feature; update.sh is required before the other four features exist for ablation

## Evidence

- [abstract] Five Nanoident features listed: "For each targeted motif sequence, Nanoident utilizes five different features, including statistical features extracted from both the nanopore raw signals and the basecalling results of the motif."
- [readme] Feature extension via update.sh: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score)."
- [methods] pkl conversion step: "we need to convert it to a pkl file that can be used in a python environment using data_from_R_to_python.ipynb"
- [methods] Ablation training/eval scripts: "python ./model/train_loocv_model.py"
- [abstract] LOOCV benchmark dataset: "The Leave-One-Out Cross-Validation (LOOCV) on the dataset containing 7 bacteria samples with 46 methylation motifs shows that Nanoident achieves ∼10% improvement in accuracy than the previous method."
