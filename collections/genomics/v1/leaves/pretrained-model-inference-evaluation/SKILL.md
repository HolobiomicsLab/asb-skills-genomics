---
name: pretrained-model-inference-evaluation
description: Use when a released pretrained checkpoint (e.g., _model/Nanoident.pt)
  must be evaluated on an independent test set with target accuracy values, and when
  a post-processing module's contribution must be quantified separately from raw model
  inference.
when_to_use_negative:
- Input requires de novo training or LOOCV cross-validation on the 7-bacteria/46-motif
  training set (use train_loocv_model.py / train_loocv_test.py instead of the pretrained
  checkpoint)
- The task is preprocessing nanopore signals or motif enrichment rather than evaluating
  model inference on an existing test pkl
- The goal is modified-base calling from modBAM files (e.g., modkit pileup) rather
  than methylation motif typing and fine mapping
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: Nanoident (pretrained model best_model/Nanoident.pt)
  role: Pretrained multi-column CNN checkpoint loaded for inference on the 12-motif
    independent test set without retraining
  repo: https://github.com/cz-csu/Nanoident
- name: postprocess.py
  role: Applies the position-correction module to raw predictions, raising accuracy
    from 73.58% to 79.92%
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Runtime environment (conda env from Nanoident/env/freeze.yml) in which train_test.py
    and postprocess.py are executed
- name: nanodisco
  role: Previous method used to compute the 66.58% baseline accuracy on the same independent
    test set
  repo: https://github.com/fanglab/nanodisco
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:48624008a7b18395e56276d173a3e87041f10c84bfa505a7c8a79d2fd9122a6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Nanoident (pretrained model best_model/Nanoident.pt)
  - Python
  - nanodisco
  - postprocess.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pretrained-model-inference-evaluation

## Summary

Evaluate a pretrained deep-learning model on an independent held-out dataset by loading the released checkpoint, running inference, optionally applying post-processing modules, and computing accuracy against published targets and baselines. This skill reproduces the Nanoident evaluation where the pretrained model achieves 73.58% (no post-processing) and 79.92% (with position-correction) on 12 independent methylation motifs.

## When to use

Use when a released pretrained checkpoint (e.g., best_model/Nanoident.pt) must be evaluated on an independent test set with target accuracy values, and when a post-processing module's contribution must be quantified separately from raw model inference.

## When NOT to use

- Input requires de novo training or LOOCV cross-validation on the 7-bacteria/46-motif training set (use train_loocv_model.py / train_loocv_test.py instead of the pretrained checkpoint)
- The task is preprocessing nanopore signals or motif enrichment rather than evaluating model inference on an existing test pkl
- The goal is modified-base calling from modBAM files (e.g., modkit pileup) rather than methylation motif typing and fine mapping

## Inputs

- Pretrained model checkpoint best_model/Nanoident.pt
- Test pkl file for 12 independent methylation motifs (6 4mC, 1 5mC, 5 6mA) from N. otitidiscaviarum NEB252 and T. phaeum DSM 12270
- Nanopore sequencing data from SRA BioProject PRJNA559199
- Reference sequences (NCBI CP041693, CP041696) and REBASE methylation motif annotations

## Outputs

- Raw methylation typing and methylated-base-position predictions on the 12 independent motifs
- test_accuracy_results.csv containing raw accuracy (73.58%), post-processed accuracy (79.92%), nanodisco baseline (66.58%), and improvement (+7.2%)
- Post-processed predictions from postprocess.py position-correction module

## How to apply

Load the pretrained multi-column CNN from best_model/Nanoident.pt rather than retraining, then run the test-results section of ./model/train_test.py on the test pkl containing the 12 independent motifs (6 4mC, 1 5mC, 5 6mA) derived from SRA BioProject PRJNA559199 to obtain raw methylation typing and methylated-base-position predictions, targeting 73.58% overall accuracy. Next run postprocess.py to apply the position-correction module to the raw predictions and recompute accuracy, targeting 79.92% (+6.34% from post-processing). Compute the nanodisco baseline on the same test set (target 66.58%) to verify the reported ~7.2% absolute (~13% relative) improvement. Validate by checking that all four accuracy values (73.58%, 79.92%, 66.58%, +7.2%) appear in test_accuracy_results.csv within a small tolerance.

## Related tools

- **Nanoident (pretrained model best_model/Nanoident.pt)** (Pretrained multi-column CNN checkpoint loaded for inference on the 12-motif independent test set without retraining) — https://github.com/cz-csu/Nanoident
- **postprocess.py** (Applies the position-correction module to raw predictions, raising accuracy from 73.58% to 79.92%) — https://github.com/cz-csu/Nanoident
- **Python** (Runtime environment (conda env from Nanoident/env/freeze.yml) in which train_test.py and postprocess.py are executed)
- **nanodisco** (Previous method used to compute the 66.58% baseline accuracy on the same independent test set) — https://github.com/fanglab/nanodisco

## Examples

```
conda env create -f freeze.yml; python ./model/train_test.py; python postprocess.py
```

## Evaluation signals

- Raw inference accuracy on the 12-motif test set matches 73.58% within a small tolerance
- Post-processed accuracy after running postprocess.py matches 79.92% (+6.34% over raw predictions)
- nanodisco baseline on the same test set matches 66.58%, confirming ~7.2% absolute (~13% relative) improvement
- All four values (73.58%, 79.92%, 66.58%, +7.2%) are present in test_accuracy_results.csv
- Test set composition confirms 12 motifs: 6 4mC, 1 5mC, 5 6mA across N. otitidiscaviarum NEB252 and T. phaeum DSM 12270

## Limitations

- Preprocessing requires preprocessing in chunks 'because of memory constraints', so large datasets cannot be processed in a single pass
- The pretrained checkpoint evaluation targets two bacterial species with exactly 12 motifs; accuracy on other species or motif distributions is not guaranteed
- nanodisco baseline requires a separate Singularity (v3.2.1 and above) environment and installation of nanodisco plus bam-readcount

## Evidence

- [results] On the independent test set of 12 motifs, Nanoident increased the overall average accuracy to 73.58% from nanodisco's 66.58%, and adding the position-correction post-processing module further increased accuracy to 79.92% (+6.34%).: [evidence span withheld — anchor status: not_found]
- [supplementary] you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [supplementary] There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs.: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
- [abstract] Furthermore, Nanoident achieves ∼13% improvement in accuracy in an independent dataset, which contains 12 methylation motifs.: "Nanoident achieves ∼13% improvement in accuracy in an independent dataset, which contains 12 methylation motifs."
- [readme] Use Nanoident/env/freeze.yml to generate the environment directly under conda.: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
