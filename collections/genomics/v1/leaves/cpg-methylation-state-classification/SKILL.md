---
name: cpg-methylation-state-classification
description: Use when calling single-molecule CpG methylation states from ONT R10.4
  nanopore signal (POD5) with a BiLSTM model, or when Dorado's calls prove unstable
  across heterogeneous human/plant datasets.
when_to_use_negative:
- Calling modification types or contexts other than CpG methylation — DeepBAM is a
  CpG methylation caller.
- Data from chemistries other than R10.4 — the paper notes only a small number of
  tools (DeepMod2, Dorado) were compatible with R10.4 and DeepBAM's models are trained/evaluated
  on R10.4 human and plant datasets.
- Input is already a methylation-frequency table or BS-seq dataset — this skill classifies
  methylation from raw nanopore signal, not from existing frequency calls.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: DeepBAM
  role: Tool implementing the seq-LSTM/signal-LSTM/combined-BiLSTM CpG methylation
    model being reconstructed and applied
  repo: github.com/xiaochuanle/DeepBAM
- name: PyTorch
  role: Framework for defining the nn.Module BiLSTM blocks and training via py/train_lstm.py
- name: Dorado
  role: Official ONT methylation caller used as the baseline DeepBAM is benchmarked
    against on R10.4 CpG datasets
- name: pod5-file-format
  role: C++ ABI for reading nanopore POD5 read signal during feature extraction
  repo: https://github.com/nanoporetech/pod5-file-format
- name: cnpy
  role: C++ library to read/write .npy/.npz feature arrays exchanged with the methylation
    model
  repo: github.com/rogersce/cnpy
- name: ThreadPool
  role: C++11 thread pool enabling asynchronous feature extraction during modification
    calling
  repo: github.com/progschj/ThreadPool
- name: argparse
  role: Argument parsing for DeepBAM's command-line interface
  repo: github.com/p-ranav/argparse
- name: spdlog
  role: Fast C++ logging library used within DeepBAM
  repo: github.com/gabime/spdlog
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:1be1610756835ef9ec5bbdcdb78f9e2b1f33dadc33138cd6d41fc8141b0c6d13
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  - PyTorch
  - ThreadPool
  - argparse
  - cnpy
  - numpy
  - pod5-file-format
  - spdlog
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# cpg-methylation-state-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BiLSTM-based single-molecule CpG methylation calling for Oxford Nanopore R10.4 reads, as implemented in DeepBAM: parallel sequence- and signal-feature BiLSTM streams are fused in a combined BiLSTM to emit a softmax methylated/unmethylated probability per CpG (avg AUROC 98.47%).

## When to use

Use when calling single-molecule CpG methylation states from ONT R10.4 nanopore signal (POD5) with a BiLSTM model, or when Dorado's calls prove unstable across heterogeneous human/plant datasets.

## When NOT to use

- Calling modification types or contexts other than CpG methylation — DeepBAM is a CpG methylation caller.
- Data from chemistries other than R10.4 — the paper notes only a small number of tools (DeepMod2, Dorado) were compatible with R10.4 and DeepBAM's models are trained/evaluated on R10.4 human and plant datasets.
- Input is already a methylation-frequency table or BS-seq dataset — this skill classifies methylation from raw nanopore signal, not from existing frequency calls.

## Inputs

- ONT R10.4 nanopore reads with raw signal in POD5 format
- Reference genome for locating CpG site windows
- High-confidence bisulfite (BS-seq) methylation labels for supervised training
- Per-site feature windows (51-mer context: base encoding, signal mean, signal std, signal length)

## Outputs

- Per-CpG methylated/unmethylated softmax probability (single-molecule methylation states)
- Modification calls produced by extract_and_call_mods
- Trained BiLSTM model checkpoint from py/train_lstm.py
- Whole-genome CpG methylation frequency tracks comparable to BS-seq

## How to apply

First extract training features with extract_hc_sites, which builds per-site windows (default kmer_size=51) encoding base features (base encoding, signal mean, signal std, signal length) plus nanopore signal features, using high-confidence bisulfite (BS-seq) labels. Train the network with py/train_lstm.py in PyTorch: two parallel two-layer BiLSTM blocks process the sequence-feature stream and the signal-feature stream, their outputs are concatenated and passed through a three-layer combined BiLSTM, and a fully connected layer with softmax outputs the methylation probability for the central CpG, optimized with cross-entropy loss. For inference, run extract_and_call_mods, which asynchronously extracts features and passes them to the trained model to emit modification calls. Validate the resulting model by comparing single-molecule AUROC/F1 against bisulfite-derived truth (paper reports 98.47% average AUROC, 94.97% average F1) and by correlating whole-genome CpG methylation frequencies with BS-seq, expecting correlations >0.95.

## Related tools

- **DeepBAM** (Tool implementing the seq-LSTM/signal-LSTM/combined-BiLSTM CpG methylation model being reconstructed and applied) — github.com/xiaochuanle/DeepBAM
- **PyTorch** (Framework for defining the nn.Module BiLSTM blocks and training via py/train_lstm.py)
- **Dorado** (Official ONT methylation caller used as the baseline DeepBAM is benchmarked against on R10.4 CpG datasets)
- **pod5-file-format** (C++ ABI for reading nanopore POD5 read signal during feature extraction) — https://github.com/nanoporetech/pod5-file-format
- **cnpy** (C++ library to read/write .npy/.npz feature arrays exchanged with the methylation model) — github.com/rogersce/cnpy
- **ThreadPool** (C++11 thread pool enabling asynchronous feature extraction during modification calling) — github.com/progschj/ThreadPool
- **argparse** (Argument parsing for DeepBAM's command-line interface) — github.com/p-ranav/argparse
- **spdlog** (Fast C++ logging library used within DeepBAM) — github.com/gabime/spdlog

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git && python py/train_lstm.py
```

## Evaluation signals

- Forward pass on a 51-mer feature window emits a valid softmax probability vector over the methylated/unmethylated classes (probabilities sum to 1).
- Single-molecule AUROC reaches the paper's reported range (98.47% average, up to 7.36% improvement over Dorado; stable ≥0.956 AUCs across datasets).
- F1 scores average 94.97% (up to 16.24% improvement over Dorado) across evaluation datasets.
- Whole-genome CpG methylation frequencies correlate >0.95 with BS-seq on 4 of 5 datasets, outperforming Dorado in all instances.
- Training via py/train_lstm.py on extract_hc_sites features converges under cross-entropy loss with high-confidence bisulfite labels.

## Limitations

- Only 9-kmer modules have been updated/released so far; other kmer modules were pending at the time of the README.
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance.
- No software version/release identifier, no changelog, and no accession IDs for the three plant or human cell line R10.4 datasets are provided in the text.
- Quantitative speed/memory comparison details are deferred to Supplementary Table 3, not present in the main text.
- Dorado's methylation model performance differs significantly across heterogeneous human and plant datasets, so cross-dataset robustness must be checked for any model, including DeepBAM.

## Evidence
