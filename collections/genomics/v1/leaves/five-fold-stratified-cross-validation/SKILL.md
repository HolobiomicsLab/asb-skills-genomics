---
name: five-fold-stratified-cross-validation
description: Use when comparing several CNN architectures for splice site prediction
  on fixed-length (400 nt) one-hot-encoded genomic sequences and you need a fair,
  class-stratified ranking by average validation accuracy to select top models.
when_to_use_negative:
- Your input sequences do not conform to the dataset's fixed sequence length of 400,
  since the CNNSplice pipeline is built for that constraint
- You need raw genomic sequence classification without one-hot encoding or model-architecture
  comparison — this skill is specifically a model-selection cross-validation procedure
- You already have trained final models and only want benchmark comparison against
  author-provided state-of-the-art models on a test dataset
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: CNNSplice
  role: Training pipeline whose train.py script implements the five-fold StratifiedKFold
    cross-validation over the 11 candidate architectures and writes .h5 models and
    .txt evaluation metrics
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: StratifiedKFold
  role: Machine learning module used to split the training dataset into five folds
    preserving the true/false splice-site class distribution
- name: Docker
  role: Runs the CNNSplice containerized environment (oluwadarelab/cnnsplice image)
    with the working directory mounted for training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:68d4c74d4882bbe3fcaac8f83e5396d87b3616ca238a472cb42628edcd8b1304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - Docker
  - StratifiedKFold
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# five-fold-stratified-cross-validation

## Summary

A five-fold stratified cross-validation procedure used by CNNSplice to rank 11 candidate CNN architectures (C1 through AVEPOOL12) by mean validation accuracy on balanced and imbalanced acceptor/donor splice-site datasets across five organisms, yielding the selection of Models 1-5.

## When to use

Use when comparing several CNN architectures for splice site prediction on fixed-length (400 nt) one-hot-encoded genomic sequences and you need a fair, class-stratified ranking by average validation accuracy to select top models.

## When NOT to use

- Your input sequences do not conform to the dataset's fixed sequence length of 400, since the CNNSplice pipeline is built for that constraint
- You need raw genomic sequence classification without one-hot encoding or model-architecture comparison — this skill is specifically a model-selection cross-validation procedure
- You already have trained final models and only want benchmark comparison against author-provided state-of-the-art models on a test dataset

## Inputs

- One-hot-encoded splice-site sequences of length 400 (balanced: 5000 true / 5000 false; imbalanced: 7500 true / 2500 false) for acceptor and donor sites
- Datasets from five organisms: Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans
- 11 candidate CNN architecture definitions (C1 through AVEPOOL12)

## Outputs

- .h5 model files per trained architecture and fold
- .txt evaluation metrics files stored in the log directory
- Per-architecture mean cross-validation accuracy rankings (Models 1-5 selection)

## How to apply

Set up the CNNSplice environment by pulling the oluwadarelab/cnnsplice Docker image and running the container with the working directory mounted. One-hot encode the splice-site sequences (fixed sequence length of 400) in both balanced (5000 true / 5000 false) and imbalanced (7500 true / 2500 false) modes for acceptor and donor sites across the five organisms. For each of the 11 candidate architectures, run train.py with the model name and mode; the training script uses StratifiedKFold to split the training dataset into five folds. Collect the .h5 model files and .txt evaluation metric files written into the log directory, average the fold accuracies per architecture (separately for acceptor and donor), and rank architectures to reproduce the Models 1-5 selection; for balanced acceptor datasets the top three were Model 1, Model 5, and Model 3.

## Related tools

- **CNNSplice** (Training pipeline whose train.py script implements the five-fold StratifiedKFold cross-validation over the 11 candidate architectures and writes .h5 models and .txt evaluation metrics) — https://github.com/OluwadareLab/CNNSplice.git
- **StratifiedKFold** (Machine learning module used to split the training dataset into five folds preserving the true/false splice-site class distribution)
- **Docker** (Runs the CNNSplice containerized environment (oluwadarelab/cnnsplice image) with the working directory mounted for training)

## Examples

```
docker run -v ${PWD}:${PWD} -p 8050:8050 -it oluwadarelab/cnnsplice; python train.py -n "model_1_balanced" -m "balanced"
```

## Evaluation signals

- A .txt evaluation-metrics file exists in the log directory for every trained architecture
- A .h5 model file is produced for each trained architecture
- Mean cross-validation accuracy across the five folds ranks the architectures so the top selections reproduce the reported Models 1-5
- For balanced acceptor datasets, the top three ranked architectures are Model 1, Model 5, and Model 3
- Input sequences are one-hot encoded and all have sequence length 400 before training

## Limitations

- The constructed dataset permits a fixed sequence length of 400 only
- A log directory must exist beforehand for the .txt evaluation metric files to be stored
- Model 4 and Model 5 are derived models whose weights are set to the mean weight across Model 1/Model 2 and Model 1/Model 3 respectively, so their performance is coupled to the base models

## Evidence

- [results] CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module [19].: "CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module [19]."
- [results] Model 1, Model 5, and Model 3 are the top three (3) performing model architectures for balanced acceptor datasets: "Model 1, Model 5, and Model 3 are the top three (3) performing model architectures for balanced acceptor datasets"
- [results] we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced"
- [readme] Usage: To train, type in the terminal `python train.py -n "model_name" -m mode `: "Usage: To train, type in the terminal `python train.py -n "model_name" -m mode `"
- [readme] Genomic sequence input data should be transfomed using one-hot encoding.: "Genomic sequence input data should be transfomed using one-hot encoding."
