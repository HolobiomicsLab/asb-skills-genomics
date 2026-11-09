---
name: cross-entropy-binary-classifier-training
description: Use when training a nanopore methylation-calling model from POD5 R10.4
  reads plus high-confidence bisulfite labels — specifically DeepBAM's BiLSTM architecture
  trained via py/train_lstm.py with cross-entropy loss.
when_to_use_negative:
- You only need modification calling with an existing trained model (use the extract_and_call_mods
  path instead of retraining).
- No high-confidence bisulfite ground truth exists for the organism/sample, since
  the cross-entropy objective depends on these labels.
- Input is older-pore or non-CpG modification data — DeepBAM was trained and evaluated
  on R10.4 CpG methylation datasets.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: DeepBAM
  role: Repository supplying extract_hc_sites feature extraction, py/train_lstm.py
    training script, and the BiLSTM model definition
  repo: https://github.com/huicongyao/DeepBAM
- name: PyTorch
  role: Framework for implementing the seq-LSTM/signal-LSTM/combined BiLSTM blocks
    and the cross-entropy training loop
- name: numpy
  role: Array handling for the .npy/.npz feature windows read via cnpy in the C++
    extraction pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:6580feac6cbdfff0fdf809297d6ec08c44e4024510c9f512030c8f0e036f7de5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM
  tools:
  - DeepBAM
  - PyTorch
  - numpy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-entropy-binary-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains DeepBAM's CpG methylation BiLSTM model on Oxford Nanopore R10.4 signal features using high-confidence bisulfite-derived labels and a cross-entropy objective, producing a softmax binary methylated/unmethylated classifier.

## When to use

Use when training a nanopore methylation-calling model from POD5 R10.4 reads plus high-confidence bisulfite labels — specifically DeepBAM's BiLSTM architecture trained via py/train_lstm.py with cross-entropy loss.

## When NOT to use

- You only need modification calling with an existing trained model (use the extract_and_call_mods path instead of retraining).
- No high-confidence bisulfite ground truth exists for the organism/sample, since the cross-entropy objective depends on these labels.
- Input is older-pore or non-CpG modification data — DeepBAM was trained and evaluated on R10.4 CpG methylation datasets.

## Inputs

- POD5 nanopore read files from R10.4 flowcells
- High-confidence CpG sites from bisulfite sequencing data (via extract_hc_sites)
- Per-site 51-mer feature vectors: base encoding, signal mean, signal std, signal length
- Methylated/unmethylated binary labels derived from high-confidence bisulfite data

## Outputs

- Trained DeepBAM BiLSTM model checkpoint emitting a softmax probability vector for the methylated/unmethylated classes
- Trained kmer-specific modules (e.g. 9-mer modules) usable downstream for modification calling

## How to apply

First run the extract_hc_sites process to pull features for model training from high-confidence bisulfite sites; each CpG is encoded as a 51-mer window (kmer_size=51) with base encoding, signal mean, signal std, and signal length. Build two parallel two-layer BiLSTM blocks: one encoding the sequence-feature stream and one encoding the nanopore signal-feature stream, then concatenate their outputs into a three-layer combined BiLSTM fusion block. Apply a fully connected layer with softmax to emit methylation probabilities for the central CpG of the window. Train with cross-entropy loss using the py/train_lstm.py script from the DeepBAM repository, consuming the bisulfite-supervised feature set. The rationale is that bisulfite-validated labels plus separate sequence/signal encoders stabilize single-molecule performance across heterogeneous human and plant R10.4 datasets where Dorado degrades.

## Related tools

- **DeepBAM** (Repository supplying extract_hc_sites feature extraction, py/train_lstm.py training script, and the BiLSTM model definition) — https://github.com/huicongyao/DeepBAM
- **PyTorch** (Framework for implementing the seq-LSTM/signal-LSTM/combined BiLSTM blocks and the cross-entropy training loop)
- **numpy** (Array handling for the .npy/.npz feature windows read via cnpy in the C++ extraction pipeline)

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git; python py/train_lstm.py
```

## Evaluation signals

- Forward pass on a 51-mer feature window yields a valid softmax probability vector (values in [0,1], summing to 1) over methylated/unmethylated classes.
- Cross-entropy loss decreases and converges on the high-confidence bisulfite-labeled training set.
- Held-out validation AUROC approaches the paper's reported 98.47% average (with AUCs ≥0.956 across heterogeneous datasets).
- Validation F1 score nears the reported 94.97% average benchmark.
- Whole-genome methylation frequencies from the trained model correlate >0.95 with BS-seq.

## Limitations

- Labels depend on high-confidence bisulfite data availability; coverage or quality issues in BS-seq propagate into training.
- At the time of the README, only 9-mer modules had been updated/released, with other kmer modules pending.
- No software version or changelog is stated for DeepBAM, and the authors acknowledge ongoing optimization is necessary.
- Quantitative speed/memory comparisons are deferred to supplementary material not present in the main text.

## Evidence

- [abstract] DeepBAM training optimization: [evidence span withheld — anchor status: not_found]
