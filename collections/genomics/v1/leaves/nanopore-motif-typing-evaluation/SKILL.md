---
name: nanopore-motif-typing-evaluation
description: Use when running Nanoident inference on bacterial Nanopore motif data
  and predicted methylation type (4mC/5mC/6mA) does not match the nucleotide at the
  predicted methylated position, requiring correction before accuracy evaluation.
when_to_use_negative:
- Predictions are already type-position consistent, so the correction loop has nothing
  to fix
- Working with modBAM/bedMethyl modified-base calls handled by modkit rather than
  de novo Nanoident motif typing outputs
- Evaluating on the LOOCV training set of 7 bacteria/46 motifs instead of the independent
  dataset without recalibrating the comparison
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: Nanoident
  role: Multi-column CNN producing per-motif methylation-type and methylated-position
    probability outputs that feed the correction loop
  repo: https://github.com/cz-csu/Nanoident
- name: postprocess.py
  role: Implements the iterative position-correction logic until predicted type matches
    the base at the predicted position
  repo: https://github.com/cz-csu/Nanoident
- name: Nanoident (pretrained model best_model/Nanoident.pt)
  role: Pretrained checkpoint used to run inference on the independent test data
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Environment (conda freeze.yml) in which inference and post-processing are
    executed
  repo: https://github.com/cz-csu/Nanoident
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:56c5f0f6d0ae2fe152fcd8927deaf7d91f4cc013ebc7350ff3d5a02cbd6a35db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Nanoident (pretrained model best_model/Nanoident.pt)
  - Python
  - postprocess.py
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nanopore-motif-typing-evaluation

## Summary

A post-processing position-correction module for Nanoident that resolves inconsistencies between the predicted methylation type and the predicted methylated base position in bacterial methylation motifs from Nanopore sequencing, improving motif typing and fine-mapping accuracy.

## When to use

Use when running Nanoident inference on bacterial Nanopore motif data and predicted methylation type (4mC/5mC/6mA) does not match the nucleotide at the predicted methylated position, requiring correction before accuracy evaluation.

## When NOT to use

- Predictions are already type-position consistent, so the correction loop has nothing to fix
- Working with modBAM/bedMethyl modified-base calls handled by modkit rather than de novo Nanoident motif typing outputs
- Evaluating on the LOOCV training set of 7 bacteria/46 motifs instead of the independent dataset without recalibrating the comparison

## Inputs

- Per-motif methylation-type and methylated-position probability outputs from Nanoident inference (train_test.py)
- Basecall base features for each motif candidate position (produced by the extended nanodisco preprocessing with update.sh)
- Pretrained model checkpoint best_model/Nanoident.pt
- Independent test set of 12 methylation motifs (6 4mC, 1 5mC, 5 6mA) from two bacteria

## Outputs

- Corrected methylated base position assignments (postprocess.py output)
- Motif typing and position-mapping accuracy with correction module
- Corrected-vs-uncorrected accuracy comparison (expected gain ~+6.34% on the independent set)

## How to apply

Load the pretrained checkpoint best_model/Nanoident.pt and run train_test.py on the independent test data to obtain per-motif methylation-type and methylated-position probability outputs. Parse these raw predictions together with the basecall base features (added during preprocessing via update.sh) and check, for each motif, whether the nucleotide at the predicted position is consistent with the predicted methylation type (e.g., an A for 6mA, C for 5mC, or the appropriate base for 4mC). For inconsistent predictions, iteratively reassign the methylated base to the candidate position with the next-highest predicted probability until type-position consistency is achieved, implementing this in postprocess.py. Finally, recompute motif typing and position-mapping accuracy on the 12-motif independent dataset with and without the correction module and compare the gains.

## Related tools

- **Nanoident** (Multi-column CNN producing per-motif methylation-type and methylated-position probability outputs that feed the correction loop) — https://github.com/cz-csu/Nanoident
- **postprocess.py** (Implements the iterative position-correction logic until predicted type matches the base at the predicted position) — https://github.com/cz-csu/Nanoident
- **Nanoident (pretrained model best_model/Nanoident.pt)** (Pretrained checkpoint used to run inference on the independent test data) — https://github.com/cz-csu/Nanoident
- **Python** (Environment (conda freeze.yml) in which inference and post-processing are executed) — https://github.com/cz-csu/Nanoident

## Examples

```
python ./model/train_test.py && python ./model/postprocess.py
```

## Evaluation signals

- Corrected motif typing/position-mapping accuracy on the 12-motif independent set exceeds uncorrected accuracy by approximately +6.34%
- Every corrected motif now satisfies type-position consistency: the nucleotide at the assigned position matches the predicted methylation type (4mC/5mC/6mA)
- Per-motif probability outputs are monotonic: correction only moves to candidate positions with the next-highest predicted probability, never arbitrary ones
- Re-running postprocess.py with best_model/Nanoident.pt reproduces the reported improvement on the independent dataset; investigate specific failing motifs otherwise

## Limitations

- Correction relies on basecall base features added by the modified nanodisco pipeline (update.sh); without them the consistency check cannot be performed
- Gains were validated on a small independent set of only 12 motifs from two bacteria, so the ~+6.34% improvement may vary on other data
- Memory constraints require chunked preprocessing, which must complete before post-processing can run

## Evidence

- [supplementary] The position correction module iteratively re-evaluates the site with the next-highest prediction probability until it finds a site whose base matches the predicted methylation type.: [evidence span withheld — anchor status: not_found]
- [readme] you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [readme] We also added base feature to facilitate positional correction.: "We also added base feature to facilitate positional correction."
- [supplementary] There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs.: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
