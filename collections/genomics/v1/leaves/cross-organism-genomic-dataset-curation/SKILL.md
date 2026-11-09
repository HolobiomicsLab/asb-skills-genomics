---
name: cross-organism-genomic-dataset-curation
description: Use when building true/false splice-site training or benchmark datasets
  across multiple organisms, specifically needing 1:1 (5000/5000) and 3:1 (7500/2500)
  acceptor/donor class ratios with fixed-length sequence windows.
when_to_use_negative:
- Your downstream model expects variable-length sequences — CNNSplice's constructed
  dataset permits a fixed Sequence Length of 400 only
- You need noncanonical splice-site classes; this skill only constructs true/false
  (positive/negative) acceptor and donor sets
- Your goal is single-organism deep training with large-scale data rather than controlled
  cross-organism benchmarking at 5000/7500 scale
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: CNNSplice
  role: CNN model pipeline whose training/testing scripts consume the balanced or
    imbalanced curated datasets via the -m mode flag
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: StratifiedKFold
  role: Splits the curated training dataset into five stratified folds for five-fold
    cross-validation model selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:45d2bcf39bf33ae74a4a5a0223665b7f76d7d883e52d4e2697a79a4547d7622b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - StratifiedKFold
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cross-organism-genomic-dataset-curation

## Summary

Constructs balanced and imbalanced acceptor/donor splice-site datasets from five organisms (Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, Caenorhabditis elegans) with fixed 400-nt sequence regions, as used by CNNSplice for evaluating models robust to data distribution skew.

## When to use

Use when building true/false splice-site training or benchmark datasets across multiple organisms, specifically needing 1:1 (5000/5000) and 3:1 (7500/2500) acceptor/donor class ratios with fixed-length sequence windows.

## When NOT to use

- Your downstream model expects variable-length sequences — CNNSplice's constructed dataset permits a fixed Sequence Length of 400 only
- You need noncanonical splice-site classes; this skill only constructs true/false (positive/negative) acceptor and donor sets
- Your goal is single-organism deep training with large-scale data rather than controlled cross-organism benchmarking at 5000/7500 scale

## Inputs

- Splice-site datasets for Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans (as used by Albaradei et al. 2020)
- True and false acceptor/donor splice-site annotations per organism
- Genomic sequences for extracting 400-nt regions around splice sites

## Outputs

- Balanced acceptor/donor datasets with 5000 true and 5000 false instances per organism
- Imbalanced acceptor/donor datasets with 7500 true and 2500 false instances per organism
- One-hot encoded 400-nt sequence matrices ready for CNN training/testing
- Summary table mirroring Table 1 (e.g., dataset_table1.csv) of per-organism counts

## How to apply

Download the splice-site datasets for the five organisms used by Albaradei et al. (2020) — Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans. For each organism, sample true and false splice-site examples to build balanced acceptor and donor sets of 5000 true and 5000 false instances (1:1 ratio), and imbalanced sets of 7500 true and 2500 false instances (3:1 ratio), mirroring Table 1. Extract 400-nt sequence regions (positions 0-399) around each splice site, respecting the fixed Sequence Length of 400 constraint. Transform genomic sequences into one-hot encoding before model input. Assemble per-organism counts into a summary table (e.g., dataset_table1.csv) mirroring Table 1, then validate that every entry has the correct class counts and 400-nt length.

## Related tools

- **CNNSplice** (CNN model pipeline whose training/testing scripts consume the balanced or imbalanced curated datasets via the -m mode flag) — https://github.com/OluwadareLab/CNNSplice.git
- **StratifiedKFold** (Splits the curated training dataset into five stratified folds for five-fold cross-validation model selection)

## Examples

```
python train.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Each organism's balanced acceptor and donor sets contain exactly 5000 true and 5000 false entries (1:1 ratio), as reported in Table 1
- Each organism's imbalanced acceptor and donor sets contain exactly 7500 true and 2500 false entries (3:1 ratio)
- All extracted sequence regions are exactly 400 nt in length (positions 0-399 around each splice site)
- Input genomic sequences are successfully transformed via one-hot encoding without dimension errors before training
- Summary table reproduces the reported Table 1 composition across all five organisms

## Limitations

- The constructed dataset permits a fixed Sequence Length of 400; other window sizes are not supported by the CNNSplice pipeline
- Genomic sequence input data must be transformed using one-hot encoding before use
- Class counts are fixed at 5000/5000 and 7500/2500 — re-sampling to other ratios departs from the published experimental design

## Evidence

- [results] we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced"
- [results] the five organisms' datasets: Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans organisms: [evidence span withheld — anchor status: not_found]
- [readme] Our constructed dataset permits a **Sequence Length of 400**: "Our constructed dataset permits a **Sequence Length of 400**"
- [methods] Genomic sequence input data should be transfomed using one-hot encoding.: "Genomic sequence input data should be transfomed using one-hot encoding."
- [results] CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module: "CNNSplice splits the training dataset into five folds using K-fold cross-validation with the StratifiedKFold machine learning module [19]."
