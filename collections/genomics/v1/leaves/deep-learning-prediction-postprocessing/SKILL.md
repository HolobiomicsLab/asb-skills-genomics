---
name: deep-learning-prediction-postprocessing
description: Use when a deep-learning model predicts both a methylation type and a
  methylated base position per motif, and the nucleotide at the predicted position
  is inconsistent with the predicted type (e.g. 6mA predicted at a non-adenine).
when_to_use_negative:
- Predictions whose nucleotide at the predicted position already matches the predicted
  methylation type need no correction
- Data lacking basecall base features at candidate motif positions, since the correction
  requires checking the nucleotide at each candidate position
- Tasks focused on initial motif discovery/enrichment rather than refining model position
  predictions
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: postprocess.py
  role: Implements the iterative position-correction logic that reassigns the methylated
    base to the next-highest-probability consistent position
  repo: https://github.com/cz-csu/Nanoident
- name: Nanoident (pretrained model best_model/Nanoident.pt)
  role: Supplies per-motif type and position probability outputs to be corrected
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Environment in which postprocess.py and the model inference are run
  repo: https://github.com/cz-csu/Nanoident
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:bb0d47396fe42c832df07014194c0f92ca189d1a36744bfe02c0dafd22da2586
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
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# deep-learning-prediction-postprocessing

## Summary

Post-processing module for Nanoident that corrects inconsistent methylation predictions by iteratively reassigning the methylated base to the candidate position with the next-highest predicted probability until the base matches the predicted methylation type (4mC/5mC/6mA).

## When to use

Use when a deep-learning model predicts both a methylation type and a methylated base position per motif, and the nucleotide at the predicted position is inconsistent with the predicted type (e.g. 6mA predicted at a non-adenine).

## When NOT to use

- Predictions whose nucleotide at the predicted position already matches the predicted methylation type need no correction
- Data lacking basecall base features at candidate motif positions, since the correction requires checking the nucleotide at each candidate position
- Tasks focused on initial motif discovery/enrichment rather than refining model position predictions

## Inputs

- Per-motif methylation-type and methylated-position probability outputs from Nanoident inference (train_test.py)
- Basecall base features for each candidate motif position (from bam-readcount / updated nanodisco preprocessing)
- Pretrained model checkpoint best_model/Nanoident.pt

## Outputs

- Corrected methylated-position assignments consistent with predicted methylation types (postprocess.py output)
- Corrected versus uncorrected motif typing and position-mapping accuracy on the 12-motif independent dataset, with the recorded gain (+6.34%)

## How to apply

Load the pretrained checkpoint best_model/Nanoident.pt and run train_test.py on the independent test set to obtain per-motif methylation-type and methylated-position probability outputs. Parse the raw predictions together with basecall base features (the base feature added in update.sh to facilitate positional correction) and check, for each of the 12 test motifs, whether the nucleotide at the predicted position matches the predicted methylation type. For inconsistent predictions, implement the iterative reassignment in postprocess.py: reassign the methylated base to the candidate position with the next-highest predicted probability and repeat until type-position consistency is achieved. Recompute typing and position-mapping accuracy on the 12-motif independent dataset with and without the correction; the corrected accuracy must exceed the uncorrected one by approximately the reported +6.34%.

## Related tools

- **postprocess.py** (Implements the iterative position-correction logic that reassigns the methylated base to the next-highest-probability consistent position) — https://github.com/cz-csu/Nanoident
- **Nanoident (pretrained model best_model/Nanoident.pt)** (Supplies per-motif type and position probability outputs to be corrected) — https://github.com/cz-csu/Nanoident
- **Python** (Environment in which postprocess.py and the model inference are run) — https://github.com/cz-csu/Nanoident

## Examples

```
python ./model/train_test.py  # then: python ./postprocess.py  (with best_model/Nanoident.pt) to apply position correction
```

## Evaluation signals

- Corrected accuracy on the independent 12-motif dataset exceeds uncorrected accuracy by approximately +6.34%; otherwise inspect failing motifs
- Every output motif assignment passes the type-position consistency check (predicted base matches predicted methylation type: 4mC->C, 5mC->C, 6mA->A)
- Running postprocess.py with the provided best_model/Nanoident.pt reproduces the reported best results
- The correction loop terminates, consuming candidate positions in strictly decreasing predicted-probability order

## Limitations

- Correction only swaps among candidate positions within the same motif's predicted probability ranking; if no candidate position matches the predicted type, consistency cannot be achieved
- Accuracy gain was validated on the 12-motif independent set (6 4mC, 1 5mC, 5 6mA); behavior on other motif compositions is not reported
- Requires the base feature added by update.sh; the original nanodisco pipeline only handled mean features and cannot supply this input

## Evidence

- [supplementary] you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [readme] We also added base feature to facilitate positional correction.: "We also added base feature to facilitate positional correction."
- [readme] The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score).: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score)."
- [abstract] Nanoident achieves ∼13% improvement in accuracy in an independent dataset, which contains 12 methylation motifs.: "Nanoident achieves ∼13% improvement in accuracy in an independent dataset, which contains 12 methylation motifs."
