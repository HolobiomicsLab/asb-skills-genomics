---
name: transformer-architecture-implementation
description: Use when implementing or retraining a transformer-based simulator that
  maps nucleotide k-mer sequences to raw nanopore signal values, e.g. from uncalled4/f5c
  eventalign training chunks.
when_to_use_negative:
- The goal is only to run inference and generate simulated signals from an existing
  FASTA/FASTQ input — use seq2squiggle predict with pretrained weights instead of
  retraining the architecture
- Input data are static per-k-mer pore models rather than segmented signal-to-reference
  eventalign data, since the model learns sequential context from segmented signal
  data, not static k-mer models
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: The transformer-based simulator whose FFT encoder/decoder, duration predictor,
    and noise module are implemented and trained; trainable on uncalled4/f5c eventalign
    data
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: uncalled4
  role: Generates event-level signal alignments (events.tsv) used as training data
    for the architecture
  repo: https://github.com/skovaka/uncalled4
- name: Python
  role: Implementation language for building and training the PyTorch model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:19cf4166ba5fa6065f6b686f437c6e80cf2a46db759ef0085dd3cbf3d000f43c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - seq2squiggle
  - uncalled4
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# transformer-architecture-implementation

## Summary

Implement the seq2squiggle non-autoregressive feed-forward transformer (FFT) architecture in PyTorch that maps one-hot encoded DNA k-mer chunks to nanopore raw signals, training it end-to-end with a combined loss of signal MSE, duration negative log-likelihood, and noise MSE.

## When to use

Use when implementing or retraining a transformer-based simulator that maps nucleotide k-mer sequences to raw nanopore signal values, e.g. from uncalled4/f5c eventalign training chunks.

## When NOT to use

- The goal is only to run inference and generate simulated signals from an existing FASTA/FASTQ input — use seq2squiggle predict with pretrained weights instead of retraining the architecture
- Input data are static per-k-mer pore models rather than segmented signal-to-reference eventalign data, since the model learns sequential context from segmented signal data, not static k-mer models

## Inputs

- uncalled4 eventalign training chunks (events.tsv) covering 16 k-mers segmented to at most 250 signal points
- One-hot encoded nucleotide k-mer sequences
- Aligned raw signal samples mapped to reference k-mers from a signal-to-reference alignment

## Outputs

- seq2squiggle_model.pt trained model checkpoint
- loss_curve.png per-epoch combined training loss plot
- Trained FFT encoder/decoder, gamma duration predictor, and Gaussian noise module weights

## How to apply

Load eventalign training chunks produced by uncalled4 (or f5c), each covering 16 k-mers segmented to at most 250 signal points, and one-hot embed the nucleotide k-mer sequences. Build the non-autoregressive model: an FFT encoder with 2 blocks, a length regulator, and an FFT decoder with 2 blocks. Add a duration predictor whose output parameterizes a gamma distribution for sampling per-k-mer signal lengths, plus a Gaussian noise module for signal-level stochasticity. Train end-to-end in PyTorch by minimizing the combined objective of signal MSE, duration NLL (tau = 0.0005), and noise MSE; the duration sampler and noise sampler correspond to the per-k-mer dwell-time and amplitude-variation components exposed as seq2squiggle's --duration-sampling and --noise-sampling options. Save trained weights to seq2squiggle_model.pt and record per-epoch combined loss for a convergence plot.

## Related tools

- **seq2squiggle** (The transformer-based simulator whose FFT encoder/decoder, duration predictor, and noise module are implemented and trained; trainable on uncalled4/f5c eventalign data) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **uncalled4** (Generates event-level signal alignments (events.tsv) used as training data for the architecture) — https://github.com/skovaka/uncalled4
- **Python** (Implementation language for building and training the PyTorch model)

## Examples

```
uncalled4 align input.ref input.slow5 --bam-in input.bam --eventalign-out events.tsv --eventalign-flags print-read-names,signal-index,samples --pore-model dna_r10.4.1_400bps_9mer
```

## Evaluation signals

- The saved checkpoint seq2squiggle_model.pt loads without error in PyTorch
- loss_curve.png shows monotone-ish convergence of the combined training loss (signal MSE + duration NLL at tau=0.0005 + noise MSE) on uncalled4 eventalign chunks
- Each training chunk contains 16 k-mers with at most 250 signal points after segmentation
- Duration predictor outputs parameterize a gamma distribution and noise module outputs a Gaussian, matching the sampler options exposed in the seq2squiggle CLI

## Limitations

- Testing workflow is not implemented in the seq2squiggle repository, so correctness must be judged from checkpoint loading and loss-curve convergence
- Training requires eventalign data from a signal-to-reference alignment (uncalled4 or f5c); uncalled4 >= 4.1.0 outputs normalized pore model values rather than scaled pA values, requiring optional transformation

## Evidence

- [abstract] Non-autoregressive transformer design: "seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences"
- [readme] Training data source: [evidence span withheld — anchor status: external_doc]
- [readme] Samplers implemented in the architecture: "Noise Sampler**: predicts per-k-mer signal amplitude variation
- **Duration Sampler**: predicts per-k-mer dwell time (event length) variation"
- [other] Training chunks specification: [evidence span withheld — anchor status: not_found]
- [other] Combined training objective: [evidence span withheld — anchor status: not_found]
