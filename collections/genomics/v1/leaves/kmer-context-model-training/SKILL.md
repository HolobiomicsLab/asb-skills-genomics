---
name: kmer-context-model-training
description: Use when tuning DeepBAM feature extraction for R10.4 nanopore CpG methylation
  calling and you must decide which k-mer context size balances AUC against feature-extraction
  and calling run-time cost.
when_to_use_negative:
- You only need to call modifications on new data with an existing trained module
  — use extract_and_call_mods rather than re-training per k-mer
- Your platform data are pre-R10.4; the article notes only a small number of tools
  (DeepMod2, Dorado) are compatible with R10.4 and DeepBAM was trained/evaluated on
  R10.4 datasets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: extract_hc_sites mode extracts signal features per k-mer context; py/train_lstm.py
    trains the Bi-LSTM model; traced_script_module exports TorchScript callers
  repo: github.com/huicongyao/DeepBAM
- name: PyTorch
  role: CUDA-enabled deep-learning framework (torch 2.0.1, CUDA 11.8) used to train
    the Bi-LSTM models and export TorchScript modules
- name: Dorado
  role: ONT base/methylation caller whose pipeline context (POD5, sorted BAM) supplies
    inputs; serves as the performance baseline DeepBAM ablation is compared against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:984f95629a95c8a0a1977bbf09be2fc50d6e11ce16a4a14cb1ffa75dd5e1557d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kmer-context-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ablate k-mer context length (13/21/51/101 bp) when extracting nanopore signal features for CpG methylation calling with DeepBAM, training a Bi-LSTM model per k-mer length to quantify the accuracy/compute trade-off (51-mer chosen as default).

## When to use

Use when tuning DeepBAM feature extraction for R10.4 nanopore CpG methylation calling and you must decide which k-mer context size balances AUC against feature-extraction and calling run-time cost.

## When NOT to use

- You only need to call modifications on new data with an existing trained module — use extract_and_call_mods rather than re-training per k-mer
- Your platform data are pre-R10.4; the article notes only a small number of tools (DeepMod2, Dorado) are compatible with R10.4 and DeepBAM was trained/evaluated on R10.4 datasets

## Inputs

- POD5 signal files (e.g. HG002 and B. vulgaris) with sorted BAM alignments and reference genomes
- BS-seq-derived positive (fully methylated) and negative (fully unmethylated) high-confidence CpG site lists at >=20x coverage
- kmer_size values 13, 21, 51, 101 with motif_type CG

## Outputs

- Per-k-mer extracted signal features as ${pod5filename}.npz files with site information and data arrays
- Per-k-mer trained Bi-LSTM models exported as TorchScript modules into traced_script_module
- A results table of test AUC and feature-extraction/modification-calling run-time per k-mer length (13, 21, 51, 101)

## How to apply

Run DeepBAM's extract_hc_sites mode on POD5 files with sorted BAM alignments, reference genomes, and BS-seq-derived fully-methylated/fully-unmethylated high-confidence CpG site lists at >=20x coverage, repeating extraction for kmer_size 13, 21, 51, and 101 (default 51) with motif_type CG; features are stored as ${pod5filename}.npz site/data arrays. Train the Bi-LSTM model per k-mer length with py/train_lstm.py under 10-fold cross-validation in a CUDA-enabled PyTorch environment (torch 2.0.1, CUDA 11.8), then export each model as a TorchScript module into traced_script_module. Record held-out test AUC and run-time per configuration; expect AUCs around 0.9560/0.9647/0.9750/0.9793 for k=13/21/51/101, where the 101-mer gains only ~0.43% AUC over the 51-mer but costs almost 78% more time, justifying the 51-mer default.

## Related tools

- **DeepBAM** (extract_hc_sites mode extracts signal features per k-mer context; py/train_lstm.py trains the Bi-LSTM model; traced_script_module exports TorchScript callers) — github.com/huicongyao/DeepBAM
- **PyTorch** (CUDA-enabled deep-learning framework (torch 2.0.1, CUDA 11.8) used to train the Bi-LSTM models and export TorchScript modules)
- **Dorado** (ONT base/methylation caller whose pipeline context (POD5, sorted BAM) supplies inputs; serves as the performance baseline DeepBAM ablation is compared against)

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git && python py/train_lstm.py
```

## Evaluation signals

- Each k-mer model's held-out test AUC is reported and approaches the paper's performance level (mean AUROC ~97.80% across datasets; k=13/21/51/101 give ~0.9560/0.9647/0.9750/0.9793)
- The AUC/runtime table contains exactly one row per k-mer length: 13, 21, 51, 101
- Each k-mer configuration produces a ${pod5filename}.npz feature file with site information and data arrays
- Each trained model exports as a TorchScript module into traced_script_module, one per k-mer
- Training runs under 10-fold cross-validation in the CUDA-enabled PyTorch environment (torch 2.0.1, CUDA 11.8)

## Limitations

- Larger k-mer contexts (e.g. 101) gain only ~0.43% AUC over the 51-mer but cost almost 78% more time, so long contexts are inefficient
- Quantitative speed/memory comparisons are deferred to Supplementary Table 3, which is not present in the main text
- The authors note ongoing optimization of DeepBAM is necessary, implying current performance is not final

## Evidence

- [results] AUC values across k-mer lengths: [evidence span withheld — anchor status: not_found]
