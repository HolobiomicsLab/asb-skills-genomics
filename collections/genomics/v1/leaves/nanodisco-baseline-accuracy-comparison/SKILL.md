---
name: nanodisco-baseline-accuracy-comparison
description: Use when evaluating a Nanopore methylation typing or fine-mapping method
  against the nanodisco baseline on the 12-motif independent dataset (SRA BioProject
  PRJNA559199), or when reproducing the reported 66.58% nanodisco accuracy.
when_to_use_negative:
- When the goal is de novo discovery of novel methylation motifs or metagenomic methylation-based
  binning rather than typing/fine-mapping accuracy benchmarking
- When the input data are not from Nanopore sequencing of bacteria with known REBASE
  motifs, since nanodisco/Nanoident are built around nanopore signal features
- When evaluating on the 7-bacteria training set — use the LOOCV evaluation (train_loocv_model.py
  / train_loocv_test.py) instead of this independent-dataset baseline
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: nanodisco
  role: Baseline method for de novo typing (6mA, 5mC, 4mC) and fine mapping of methylation
    motifs; provides the 66.58% comparison accuracy and the Singularity-based R environment
  repo: https://github.com/fanglab/nanodisco
- name: Nanoident
  role: Multi-column CNN under evaluation; loaded from best_model/Nanoident.pt to
    produce typing and base-position predictions on the 12 test motifs
  repo: https://github.com/cz-csu/Nanoident
- name: Nanoident (pretrained model best_model/Nanoident.pt)
  role: Pretrained checkpoint used to reproduce test-set predictions without retraining
  repo: https://github.com/cz-csu/Nanoident
- name: postprocess.py
  role: Applies the position-correction module to raw Nanoident predictions, lifting
    accuracy from 73.58% to 79.92%
  repo: https://github.com/cz-csu/Nanoident
- name: Singularity v3.2.1
  role: Container runtime used to build the nanodisco virtual environment following
    the fanglab/nanodisco instructions
  repo: https://github.com/fanglab/nanodisco
- name: bam-readcount
  role: Installed prior to nanodisco preprocessing to obtain basecall (readcount)
    features from alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:ae4375d64ea85a06408cb15da5ae18b165cbb09b20cc79e60a1d3d83d794fe47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/nanodisco
  tools:
  - Nanoident
  - Nanoident (pretrained model best_model/Nanoident.pt)
  - Singularity v3.2.1
  - bam-readcount
  - nanodisco
  - postprocess.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nanodisco-baseline-accuracy-comparison

## Summary

Establishes the nanodisco baseline typing/fine-mapping accuracy on an independent test set of 12 methylation motifs (N. otitidiscaviarum NEB252 and T. phaeum DSM 12270) so that Nanoident's 73.58% (no post-processing) and 79.92% (with position-correction module) accuracies can be benchmarked against it.

## When to use

Use when evaluating a Nanopore methylation typing or fine-mapping method against the nanodisco baseline on the 12-motif independent dataset (SRA BioProject PRJNA559199), or when reproducing the reported 66.58% nanodisco accuracy.

## When NOT to use

- When the goal is de novo discovery of novel methylation motifs or metagenomic methylation-based binning rather than typing/fine-mapping accuracy benchmarking
- When the input data are not from Nanopore sequencing of bacteria with known REBASE motifs, since nanodisco/Nanoident are built around nanopore signal features
- When evaluating on the 7-bacteria training set — use the LOOCV evaluation (train_loocv_model.py / train_loocv_test.py) instead of this independent-dataset baseline

## Inputs

- Independent test set pkl for 12 methylation motifs (6 4mC, 1 5mC, 5 6mA) from SRA BioProject PRJNA559199
- Pretrained Nanoident model best_model/Nanoident.pt
- nanodisco Singularity image (library://fanglab/default/nanodisco) and bam-readcount
- Reference sequences NCBI CP041693 and CP041696 (N. otitidiscaviarum NEB252, T. phaeum DSM 12270)
- Methylation motifs from the REBASE database

## Outputs

- test_accuracy_results.csv containing 73.58% (no post-processing), 79.92% (with module), 66.58% (nanodisco baseline), and ~+7.2% improvement
- Raw methylation typing and methylated-base-position predictions for the 12 test motifs
- Post-processed (position-corrected) predictions from postprocess.py

## How to apply

Build the nanodisco environment as a Singularity container (v3.2.1 and above) per fanglab/nanodisco, with bam-readcount installed first. Derive the test pkl for the 12 independent motifs (6 4mC, 1 5mC, 5 6mA) from PRJNA559199, convert R output to pkl with data_from_R_to_python.ipynb, then run the test-results section of ./model/train_test.py to get raw Nanoident predictions and accuracy (target 73.58%). Run postprocess.py to apply the position-correction module and recompute accuracy (target 79.92%). Compute the nanodisco baseline on the same 12 motifs (target 66.58%), then report the improvement of Nanoident over it (~+7.2% absolute, ~13% improvement over the previous method).

## Related tools

- **nanodisco** (Baseline method for de novo typing (6mA, 5mC, 4mC) and fine mapping of methylation motifs; provides the 66.58% comparison accuracy and the Singularity-based R environment) — https://github.com/fanglab/nanodisco
- **Nanoident** (Multi-column CNN under evaluation; loaded from best_model/Nanoident.pt to produce typing and base-position predictions on the 12 test motifs) — https://github.com/cz-csu/Nanoident
- **Nanoident (pretrained model best_model/Nanoident.pt)** (Pretrained checkpoint used to reproduce test-set predictions without retraining) — https://github.com/cz-csu/Nanoident
- **postprocess.py** (Applies the position-correction module to raw Nanoident predictions, lifting accuracy from 73.58% to 79.92%) — https://github.com/cz-csu/Nanoident
- **Singularity v3.2.1** (Container runtime used to build the nanodisco virtual environment following the fanglab/nanodisco instructions) — https://github.com/fanglab/nanodisco
- **bam-readcount** (Installed prior to nanodisco preprocessing to obtain basecall (readcount) features from alignments)

## Examples

```
singularity pull --name nanodisco.sif library://fanglab/default/nanodisco; python ./model/train_test.py; python postprocess.py
```

## Evaluation signals

- nanodisco baseline accuracy on the 12-motif independent test set equals 66.58% within a small tolerance
- Nanoident accuracy without post-processing equals 73.58%; with postprocess.py it rises to 79.92% (+6.34%)
- Absolute improvement over nanodisco is ~+7.2%, consistent with the reported ~13% relative improvement
- All four values (73.58%, 79.92%, 66.58%, +7.2%) appear in test_accuracy_results.csv
- The test set composition matches the expected 12 motifs (6 4mC, 1 5mC, 5 6mA) from N. otitidiscaviarum NEB252 and T. phaeum DSM 12270

## Limitations

- Original nanodisco could only handle mean features; Nanoident adds std, time, proportion of reference bases, and average quality score via update.sh, so raw nanodisco does not exploit the richer feature set
- nanodisco preprocessing must be done in chunks due to memory constraints
- Only two bacterial species (12 motifs) are available as an independent test set, so the baseline comparison is limited in scope

## Evidence

- [abstract] On an independent dataset containing 12 methylation motifs, Nanoident achieves ~13% improvement in accuracy.: "Furthermore, Nanoident achieves ∼13% improvement in accuracy in an independent dataset, which contains 12 methylation motifs."
- [supplementary] Test set motif composition and species.: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
- [readme] nanodisco environment build instructions.: "follow the nanodisco (https://github.com/fanglab/nanodisco/tree/master) to build the virtual environment using Singularity (v3.2.1 and above)."
- [supplementary] Use of pretrained model and postprocessing for best results.: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [methods] nanodisco requires bam-readcount installation before preprocessing.: "We should install nanodisco and bam-readcount first."
