---
name: runtime-benchmarking
description: Use when you must choose a k-mer context length (or similar hyperparameter)
  for DeepBAM feature extraction by trading measured test AUC against feature-extraction
  and modification-calling run-time cost on HG002/B. vulgaris POD5 data.
when_to_use_negative:
- Your goal is simply to call methylation with a trained model rather than evaluate
  k-mer length tradeoffs — use extract_and_call_mods with the default 51-mer instead
- Your data are not ONT R10.4 nanopore reads with matched BS-seq ground truth; the
  AUC benchmarks are calibrated to R10.4 human/plant datasets with fully-methylated/unmethylated
  controls
- You need comprehensive wall-clock profiling across many hyperparameters — this skill
  covers only the four k-mer lengths and defers detailed speed/memory numbers to Supplementary
  Table 3
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: DeepBAM
  role: Performs k-mer-sized feature extraction (extract_hc_sites) and Bi-LSTM model
    training (py/train_lstm.py) for each k-mer configuration
  repo: https://github.com/huicongyao/DeepBAM
- name: PyTorch
  role: CUDA-enabled training framework (torch 2.0.1, CUDA 11.8) used to train and
    export the per-k-mer Bi-LSTM models as TorchScript modules
- name: Dorado
  role: Reference baseline methylation caller whose across-dataset performance variability
    motivates the DeepBAM model benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:f49b4662ff990b13ead9e15d8972992cfdc61b8ee23e2357df12b8cfaa38de95
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
  - Dorado
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the accuracy-versus-compute-time tradeoff of DeepBAM k-mer context lengths (13/21/51/101 bp) for CpG methylation calling on ONT R10.4 data, justifying the 51-mer default. Produces a per-k-mer table of test AUC and feature-extraction/modification-calling runtime.

## When to use

Use when you must choose a k-mer context length (or similar hyperparameter) for DeepBAM feature extraction by trading measured test AUC against feature-extraction and modification-calling run-time cost on HG002/B. vulgaris POD5 data.

## When NOT to use

- Your goal is simply to call methylation with a trained model rather than evaluate k-mer length tradeoffs — use extract_and_call_mods with the default 51-mer instead
- Your data are not ONT R10.4 nanopore reads with matched BS-seq ground truth; the AUC benchmarks are calibrated to R10.4 human/plant datasets with fully-methylated/unmethylated controls
- You need comprehensive wall-clock profiling across many hyperparameters — this skill covers only the four k-mer lengths and defers detailed speed/memory numbers to Supplementary Table 3

## Inputs

- POD5 signal files (HG002 and B. vulgaris nanopore R10.4)
- Sorted BAM alignment files against reference genomes
- BS-seq-derived fully-methylated and fully-unmethylated high-confidence CpG site lists at >=20x coverage
- k-mer size configurations: 13, 21, 51, 101 (default 51), motif_type CG

## Outputs

- Per-k-mer feature arrays stored as ${pod5filename}.npz files
- One TorchScript module per k-mer in the traced_script_module directory
- A results table with one row per k-mer length (13, 21, 51, 101) containing test AUC and feature-extraction/modification-calling run time

## How to apply

Run DeepBAM's extract_hc_sites mode repeatedly on the same POD5 inputs (HG002 and B. vulgaris) with sorted BAM alignments, reference genomes, BS-seq-derived high-confidence CpG lists at >=20x coverage, and motif_type CG, once per kmer_size (13, 21, 51, and the default 51 alternative 101). Store each run's signal features as ${pod5filename}.npz files, then train the Bi-LSTM model for each length with py/train_lstm.py under 10-fold cross-validation on a CUDA-enabled PyTorch stack (torch 2.0.1, CUDA 11.8), exporting each model as a TorchScript module into traced_script_module. Record the held-out test AUC per k-mer length alongside the timed feature-extraction and modification-calling cost of each configuration, and compile these into a single AUC/runtime table. The decision rule from the paper: the 51-mer is the default because the 101-mer gained only 0.43% AUC (0.9793 vs 0.9750) but cost almost 78% more time.

## Related tools

- **DeepBAM** (Performs k-mer-sized feature extraction (extract_hc_sites) and Bi-LSTM model training (py/train_lstm.py) for each k-mer configuration) — https://github.com/huicongyao/DeepBAM
- **PyTorch** (CUDA-enabled training framework (torch 2.0.1, CUDA 11.8) used to train and export the per-k-mer Bi-LSTM models as TorchScript modules)
- **Dorado** (Reference baseline methylation caller whose across-dataset performance variability motivates the DeepBAM model benchmark)

## Evaluation signals

- The AUC/runtime table contains exactly one row per k-mer length (13, 21, 51, 101)
- Test AUCs should increase monotonically with k-mer length, approaching 0.9560 / 0.9647 / 0.9750 / 0.9793
- The 101-mer's runtime cost should be nearly 78% higher than the 51-mer's while yielding only ~0.43% extra AUC
- Each configuration produces ${pod5filename}.npz feature files and a corresponding TorchScript module in traced_script_module
- Mean AUROC across datasets should approach the paper's reported ~97.80% average performance level

## Limitations

- The 101-mer gains only 0.43% AUC over the 51-mer at almost 78% more time, so longer contexts give diminishing returns
- Quantitative speed/memory comparison details are deferred to Supplementary Table 3 and are not present in the main text
- The benchmark is specific to ONT R10.4 data with BS-seq-derived high-confidence CpG labels at >=20x coverage; no DeepBAM software version or changelog is stated in the repository

## Evidence
