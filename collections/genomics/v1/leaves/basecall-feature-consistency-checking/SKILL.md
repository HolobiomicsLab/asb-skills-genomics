---
name: basecall-feature-consistency-checking
description: Use when a per-motif methylation-type and methylated-position prediction
  from Nanoident inference must be validated against basecall base features, e.g.
  a predicted 6mA position that is not an A in the motif sequence, to recover position-mapping
  accuracy (~+6.34%).
when_to_use_negative:
- The input is not Nanoident model output — e.g. already pileup-converted ONT modified-base
  calls (modBAM/bedMethyl via modkit), which have no per-motif position probability
  distribution to re-rank.
- The methylation type of the motif is unknown or outside 4mC/5mC/6mA, since the consistency
  rule maps each type to an expected nucleotide (A or C).
- Working on the 7-sample, 46-motif LOOCV training dataset rather than the independent
  12-motif test set, where the correction module's reported gain was measured.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Nanoident
  role: Multi-column CNN producing methylation-type and methylated-position probabilities
    that the consistency check corrects; pretrained checkpoint best_model/Nanoident.pt
    is used for inference
  repo: https://github.com/cz-csu/Nanoident
- name: postprocess.py
  role: Script implementing the iterative next-highest-probability position reassignment
    until type-position consistency is achieved
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Environment (from Nanoident/env/freeze.yml under conda) in which inference
    and the position-correction post-processing run
  repo: https://github.com/cz-csu/Nanoident
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:53c22cec4dcb88c26ad1282e5a6c76c3dd26c496fa86b6b679b408d7bfe8d1c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Python
  - postprocess.py
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# basecall-feature-consistency-checking

## Summary

A post-processing consistency check from Nanoident that verifies whether the predicted methylated position within a motif actually carries the nucleotide implied by the predicted methylation type (4mC→A, 5mC→C, 6mA→A), and if not, iteratively reassigns the methylated base to the candidate position with the next-highest predicted probability until type-position consistency is achieved.

## When to use

Use when a per-motif methylation-type and methylated-position prediction from Nanoident inference must be validated against basecall base features, e.g. a predicted 6mA position that is not an A in the motif sequence, to recover position-mapping accuracy (~+6.34%).

## When NOT to use

- The input is not Nanoident model output — e.g. already pileup-converted ONT modified-base calls (modBAM/bedMethyl via modkit), which have no per-motif position probability distribution to re-rank.
- The methylation type of the motif is unknown or outside 4mC/5mC/6mA, since the consistency rule maps each type to an expected nucleotide (A or C).
- Working on the 7-sample, 46-motif LOOCV training dataset rather than the independent 12-motif test set, where the correction module's reported gain was measured.

## Inputs

- Raw Nanoident inference outputs: per-motif methylation-type and methylated-position probability distributions from train_test.py on independent test data
- Basecall base features extracted from nanopore reads (base identity at motif positions, via nanodisco + bam-readcount preprocessing)
- Pretrained model checkpoint best_model/Nanoident.pt
- Independent test dataset of 12 methylation motifs (6 4mC, 1 5mC, 5 6mA) from two bacterial species

## Outputs

- Corrected methylated-position assignments per motif, consistent with the predicted methylation type (postprocess.py output)
- Motif typing and position-mapping accuracy on the 12-motif independent dataset, computed with and without correction
- Corrected-versus-uncorrected accuracy comparison quantifying the correction gain (~+6.34%)

## How to apply

Load the pretrained checkpoint best_model/Nanoident.pt and run inference with train_test.py on the independent test data to obtain, for each of the 12 motifs, methylation-type and methylated-position probability outputs. Parse these raw predictions together with the basecall base features (extracted via the updated nanodisco/bam-readcount pipeline, which Nanoident extended beyond mean features to std, time, proportion of reference bases, average quality score, and base identity). For each motif, check whether the nucleotide at the top-probability methylated position matches the predicted methylation type (e.g. an A for 6mA or 4mC, a C for 5mC). When inconsistent, iteratively reassign the methylated base to the candidate position holding the next-highest predicted probability until type-position consistency is reached, implementing this in postprocess.py. Finally, recompute motif typing and position-mapping accuracy on the 12-motif independent dataset with and without the correction to confirm the reported gain of approximately +6.34%.

## Related tools

- **Nanoident** (Multi-column CNN producing methylation-type and methylated-position probabilities that the consistency check corrects; pretrained checkpoint best_model/Nanoident.pt is used for inference) — https://github.com/cz-csu/Nanoident
- **postprocess.py** (Script implementing the iterative next-highest-probability position reassignment until type-position consistency is achieved) — https://github.com/cz-csu/Nanoident
- **Python** (Environment (from Nanoident/env/freeze.yml under conda) in which inference and the position-correction post-processing run) — https://github.com/cz-csu/Nanoident

## Examples

```
python ./model/train_test.py  # then apply correction: run postprocess.py with best_model/Nanoident.pt predictions to enforce type-position consistency
```

## Evaluation signals

- After correction, every output motif's methylated position nucleotide matches the predicted methylation type (4mC→A, 5mC→C, 6mA→A).
- Corrected position-mapping accuracy on the 12-motif independent dataset exceeds uncorrected accuracy by approximately +6.34%; inspect failing motifs otherwise.
- Motif typing accuracy is unchanged by the correction (correction only reassigns positions, not types).
- Reassigned positions correspond to the next-highest entries in the original position-probability distribution, not arbitrary bases.
- The pipeline runs reproducibly with the provided checkpoint best_model/Nanoident.pt and postprocess.py as documented in the supplementary.

## Limitations

- The check depends on basecall base features extracted by the modified nanodisco pipeline; the original nanodisco only handled mean features, so update.sh must be applied first or base features will be absent.
- If all candidate positions for a motif are inconsistent with the predicted type, the iterative reassignment cannot yield a consistent assignment and the motif must be inspected manually.
- The reported +6.34% gain is specific to the independent 12-motif dataset (two species); gains on other datasets are not established.

## Evidence

- [supplementary] The position correction module iteratively re-evaluates the site with the next-highest prediction probability until it finds a site whose base matches the predicted methylation type.: [evidence span withheld — anchor status: not_found]
- [supplementary] you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [readme] We also added base feature to facilitate positional correction.: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score). We also added base"
- [supplementary] There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs.: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
