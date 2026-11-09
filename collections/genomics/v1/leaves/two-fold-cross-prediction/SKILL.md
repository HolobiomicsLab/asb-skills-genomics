---
name: two-fold-cross-prediction
description: 'Use when building DeepSignal-plant training sets from Nanopore signal
  features (13-mer + 13x16 per cytosine): after per-k-mer balancing, to purge mislabelled
  methylated samples (esp. CHH/CHG) before calling `deepsignal_plant train`.'
when_to_use_negative:
- You already have a curated gold-standard training set (e.g. from an established
  plant methylation model checkpoint like model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt)
  — re-denoising adds cost without new labels
- Your data is from a different organism/non-plant context where bisulfite-derived
  high-confidence 5mC labels at the required thresholds (>=5 reads, freq >=0.9 / zero)
  cannot be obtained
- You are only calling methylation on new reads with an existing model (call_mods/call_freq
  path), not training or re-training a model
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: DeepSignal-plant
  role: Provides the BiLSTM BRNN model, Adam/cross-entropy training loop, and extract/train
    functions used for the two-fold cross prediction and sample removal
  repo: https://github.com/PengNi/deepsignal-plant
- name: PyTorch
  role: Framework (>=1.2.0, <=1.11.0) in which the BRNN-with-LSTM denoising model
    is trained and evaluated
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Resquiggles raw Nanopore signals to genome reference bases, producing the
    signal-aligned features consumed by the training samples
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:4775dc79debec9fa6921a568a5a200b1a780462b41bd91a1fe227945a4e8a293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - PyTorch
  - Tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# two-fold-cross-prediction

## Summary

Iterative denoising procedure from DeepSignal-plant that removes falsely labelled methylated (5mC) training samples by repeated two-fold cross prediction with a BRNN/LSTM model, terminating at I=10 iterations or >=99% sample retention.

## When to use

Use when building DeepSignal-plant training sets from Nanopore signal features (13-mer + 13x16 per cytosine): after per-k-mer balancing, to purge mislabelled methylated samples (esp. CHH/CHG) before calling `deepsignal_plant train`.

## When NOT to use

- You already have a curated gold-standard training set (e.g. from an established plant methylation model checkpoint like model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt) — re-denoising adds cost without new labels
- Your data is from a different organism/non-plant context where bisulfite-derived high-confidence 5mC labels at the required thresholds (>=5 reads, freq >=0.9 / zero) cannot be obtained
- You are only calling methylation on new reads with an existing model (call_mods/call_freq path), not training or re-training a model

## Inputs

- Balanced training-sample TSV produced by `deepsignal_plant extract`: per targeted cytosine a 13-mer sequence context and 13x16 per-base Nanopore signal features
- Positive (high-confidence methylated) samples: cytosines with >=5 reads and >=0.9 methylation frequency by bisulfite sequencing
- Negative (high-confidence unmethylated) samples: cytosines with >=5 mapped reads and zero bisulfite methylation frequency
- Pre-trained/checkpoint BiLSTM model config for PyTorch (e.g. epoch6.ckpt architecture)

## Outputs

- Denoised, per-k-mer balanced training file of retained methylated and unmethylated 13-mer samples with 13x16 signal features
- Iteration log of removed methylated samples per fold-prediction round (mean(score(s)) < 0.5)

## How to apply

Load the balanced training file of 13-mer sequences and 13x16 normalized-signal features (median shift + MAD scale) per targeted cytosine. Per iteration, split the current set into two folds and, for R=3 rounds, train the BiLSTM BRNN (Adam optimizer, cross-entropy loss) on one fold and cross-predict the held-out fold. Remove each positive sample s whose mean predicted methylation probability mean(score(s)) < 0.5, then re-apply Algorithm 1 balancing (per-k-mer motif balance and per-k-mer read-coverage capping). Iterate with defaults I=10, R=3, E=3, stopping early when fewer than 1% of methylated samples are removed (>=99% retention); write the retained positive/negative samples for the final `deepsignal_plant train` run.

## Related tools

- **DeepSignal-plant** (Provides the BiLSTM BRNN model, Adam/cross-entropy training loop, and extract/train functions used for the two-fold cross prediction and sample removal) — https://github.com/PengNi/deepsignal-plant
- **PyTorch** (Framework (>=1.2.0, <=1.11.0) in which the BRNN-with-LSTM denoising model is trained and evaluated) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Resquiggles raw Nanopore signals to genome reference bases, producing the signal-aligned features consumed by the training samples) — https://github.com/nanoporetech/tombo

## Evaluation signals

- After each removal round, the retained positive fraction must be computed; termination requires >=99% retention of methylated samples (i.e. <1% predicted as unmethylated) or hitting I=10 iterations
- Every removed positive sample must satisfy mean(score(s)) < 0.5 across the R=3 cross-prediction rounds; negatives are never removed
- The denoised output must remain balanced per k-mer motif and capped per-k-mer read coverage (Algorithm 1 re-applied after each removal)
- Final trained model should achieve high correlation with bisulfite sequencing across CpG, CHG, and CHH contexts, with the largest gains expected for CHH and CHG
- Feature schema check: each retained sample still carries 13-mer sequence plus 13x16 signal features consistent with the deepsignal_plant extract output

## Limitations

- Gold-standard labels derive from bisulfite sequencing, so denoising cannot correct bisulfite artefacts (incomplete conversion, degradation, missing repetitive-region coverage)
- For A. thaliana CHG/CHH, high-confidence methylated labels use the union of three replicates (lower stringency), injecting label noise that denoising only partially mitigates
- Requires PyTorch (>=1.2.0, <=1.11.0) and Tombo 1.5.1 with the ont-vbz-hdf-plugin on HDF5_PLUGIN_PATH set, otherwise resquiggle/extract steps fail on VBZ-compressed fast5s
- Run as a 20-million-sample cap (half positive, half negative) per motif; larger sets are randomly subsampled before denoising

## Evidence

- [discussion] In the denoising algorithm, we set I = 10, R = 3, E = 3 as default: "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [discussion] if mean(score(s)) < 0.5 thenSpos.remove(s): "if mean(score(s)) < 0.5 thenSpos.remove(s)"
- [discussion] we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer: "we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer"
- [readme] deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads. It is built on **Python3** and **PyTorch**."
- [discussion] raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
