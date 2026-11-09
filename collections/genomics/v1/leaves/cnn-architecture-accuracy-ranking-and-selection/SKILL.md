---
name: cnn-architecture-accuracy-ranking-and-selection
description: Use when you must select among multiple CNN architectures by mean validation
  accuracy under StratifiedKFold cross-validation, as in CNNSplice's model selection
  over 11 candidates.
when_to_use_negative:
- The task is benchmarking CNNSplice against external tools (GeneSplicer, SpliceFinder,
  etc.) on a held-out test set rather than selecting among its own candidate architectures
- Input sequences cannot be constrained to the required sequence length of 400, since
  the constructed dataset permits only that length
- Only a single architecture needs to be trained and evaluated, so no cross-architecture
  ranking or selection is required
edam_operation: http://edamontology.org/operation_0480
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: CNNSplice
  role: Training pipeline whose train.py script trains each candidate architecture
    and writes .h5 models and .txt evaluation metrics used for ranking
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: StratifiedKFold
  role: Machine learning module that splits the training dataset into five stratified
    folds for cross-validation model selection
- name: Docker
  role: Runtime environment; pull oluwadarelab/cnnsplice:latest and run the container
    with the working directory mounted to execute training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:1ae2ead5f5d2ee906a9546c723d9c6fe01a2943c656a1674c5f7f5d0d9a4c334
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - Docker
  - StratifiedKFold
  edam_operation: http://edamontology.org/operation_0480
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cnn-architecture-accuracy-ranking-and-selection

## Summary

Ranks candidate CNN architectures by averaging five-fold cross-validation accuracy on balanced splice-site datasets across five organisms, to select the top-performing models (Models 1-5) for CNNSplice.

## When to use

Use when you must select among multiple CNN architectures by mean validation accuracy under StratifiedKFold cross-validation, as in CNNSplice's model selection over 11 candidates.

## When NOT to use

- The task is benchmarking CNNSplice against external tools (GeneSplicer, SpliceFinder, etc.) on a held-out test set rather than selecting among its own candidate architectures
- Input sequences cannot be constrained to the required sequence length of 400, since the constructed dataset permits only that length
- Only a single architecture needs to be trained and evaluated, so no cross-architecture ranking or selection is required

## Inputs

- One-hot encoded genomic splice-site sequences with fixed sequence length of 400
- Balanced datasets (5000 true / 5000 false) and imbalanced datasets (7500 true / 2500 false) for acceptor and donor sites from five organisms
- Eleven candidate CNN architecture definitions (C1 through AVEPOOL12)

## Outputs

- .h5 deep learning model files per trained architecture
- .txt evaluation-metrics files stored in the log directory
- Ranked list of architectures by mean five-fold cross-validation accuracy (selection of Models 1-5)

## How to apply

Run the CNNSplice Docker container (oluwadarelab/cnnsplice:latest) with the working directory mounted, then one-hot encode fixed-length (400 nt) genomic splice-site sequences for the balanced acceptor and donor datasets of the five organisms (Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, Caenorhabditis elegans). For each of the 11 candidate architectures (C1 through AVEPOOL12), run train.py with mode 'balanced' or 'imbalanced'; the script uses StratifiedKFold to split the training data into five folds and writes .h5 model files plus .txt evaluation-metric files into the log directory. Average the cross-validation accuracies across the five folds per architecture, separately for acceptor and donor datasets, then rank architectures to reproduce the Models 1-5 selection. Confirm a .txt metrics file exists for every trained architecture and that the top three balanced-acceptor architectures are Model 1, Model 5, and Model 3. Optionally convert accuracy values to ranks (highest accuracy = rank 1) for a Friedman test of statistical significance.

## Related tools

- **CNNSplice** (Training pipeline whose train.py script trains each candidate architecture and writes .h5 models and .txt evaluation metrics used for ranking) — https://github.com/OluwadareLab/CNNSplice.git
- **StratifiedKFold** (Machine learning module that splits the training dataset into five stratified folds for cross-validation model selection)
- **Docker** (Runtime environment; pull oluwadarelab/cnnsplice:latest and run the container with the working directory mounted to execute training)

## Examples

```
python train.py -n "model_1" -m "balanced"
```

## Evaluation signals

- A .txt evaluation-metrics file exists in the log directory for every trained architecture
- Mean validation accuracy averaged over the five folds reproduces the top three balanced-acceptor architectures: Model 1, Model 5, and Model 3
- The overall ranking selects exactly Models 1-5 as the high-performing architectures
- Every one-hot encoded input sequence conforms to the fixed sequence length of 400
- For imbalanced mode, datasets contain 7500 true and 2500 false splice sites; for balanced mode, 5000 true and 5000 false

## Limitations

- The constructed dataset permits a fixed sequence length of 400; other lengths are not supported
- Ranks are computed separately for acceptor and donor datasets, so a single global ranking may not match both site types
- Cross-validation accuracies are averaged across folds; fold-level variance is not the primary selection criterion

## Evidence

- [results] Model 1, Model 5, and Model 3 are the top three (3) performing model architectures for balanced acceptor datasets: "Model 1, Model 5, and Model 3 are the top three (3) performing model architectures for balanced acceptor datasets"
- [results] CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module [19].: "CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module [19]."
- [results] we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false"
- [readme] Usage: To train, type in the terminal `python train.py -n "model_name" -m mode `: "Usage: To train, type in the terminal `python train.py -n "model_name" -m mode `"
- [readme] The outputs of training includes: .h5: The deepslicer model file. .txt: The output files (.txt) containig the evaluation metrics results is stored in the log directory.: ".txt: The output files (.txt) containig the evaluation metrics results is stored in the log directory."
