---
name: duration-distribution-sampling-loss-tuning
description: Use when training or reproducing a transformer-based nanopore signal
  simulator that needs per-k-mer dwell-time variation, or when tuning the joint loss
  (signal MSE + duration NLL + noise MSE) of a non-autoregressive sequence-to-signal
  model.
when_to_use_negative:
- The goal is inference with pre-trained seq2squiggle weights using static dwell distributions
  (--duration-sampling False --dwell-mean --dwell-std) rather than retraining the
  duration predictor
- Training data comes from a static k-mer pore model rather than signal-to-reference
  event alignments (uncalled4/f5c eventalign tables), since the duration sampler learns
  from segmented signal data
- The task is read-level simulation of synthetic reads from a reference without needing
  per-k-mer dwell-time modeling
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: seq2squiggle
  role: Deep-learning tool whose FFT-based architecture (encoder, length regulator,
    decoder, gamma duration predictor, Gaussian noise module) and combined MSE/NLL
    loss are being reconstructed and trained
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: uncalled4
  role: Generates the event-level signal-to-reference alignments (eventalign table)
    used as training chunks for the duration predictor
  repo: https://github.com/skovaka/uncalled4
- name: Python
  role: Implementation language (PyTorch) for the transformer, gamma duration sampler,
    and joint loss optimization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:72a0b7f305427322279a17cf9e0c0d7fc0821927dcd86300ad8d258f0fbfe353
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - seq2squiggle
  - uncalled4
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# duration-distribution-sampling-loss-tuning

## Summary

Reconstructs and tunes the seq2squiggle non-autoregressive transformer training objective, in which a duration predictor parameterizes a gamma distribution over per-k-mer signal lengths and is optimized jointly with signal MSE and Gaussian noise MSE losses. It is the key mechanism by which the model learns dwell-time (event length) variability from uncalled4 eventalign data instead of a static k-mer model.

## When to use

Use when training or reproducing a transformer-based nanopore signal simulator that needs per-k-mer dwell-time variation, or when tuning the joint loss (signal MSE + duration NLL + noise MSE) of a non-autoregressive sequence-to-signal model.

## When NOT to use

- The goal is inference with pre-trained seq2squiggle weights using static dwell distributions (--duration-sampling False --dwell-mean --dwell-std) rather than retraining the duration predictor
- Training data comes from a static k-mer pore model rather than signal-to-reference event alignments (uncalled4/f5c eventalign tables), since the duration sampler learns from segmented signal data
- The task is read-level simulation of synthetic reads from a reference without needing per-k-mer dwell-time modeling

## Inputs

- uncalled4 (or f5c) eventalign table (events.tsv) mapping raw signals to reference sequences, from data such as dna_r10.4.1_400bps_9mer FLO-MIN114 / SQK-LSK114 reads
- Training chunks of 16 k-mers segmented to at most 250 signal points
- One-hot encoded nucleotide k-mer sequences

## Outputs

- Trained PyTorch checkpoint (seq2squiggle_model.pt)
- Per-epoch combined-loss training curve (loss_curve.png)
- Model capable of predicting per-k-mer gamma-distributed event durations and raw nanopore signals (BLOW5/POD5 output at inference)

## How to apply

Prepare training chunks from uncalled4 eventalign tables, where each chunk covers 16 k-mers segmented to at most 250 signal points, and one-hot embed the nucleotide k-mer sequences. Build the non-autoregressive architecture: a feed-forward transformer (FFT) encoder with 2 blocks, a length regulator, and an FFT decoder with 2 blocks. Attach a duration predictor whose output parameterizes a gamma distribution from which per-k-mer signal lengths are sampled, plus a Gaussian noise module for signal-level stochasticity. Train end-to-end in PyTorch by minimizing the combined objective of signal MSE, duration negative log-likelihood (tau = 0.0005), and noise MSE. Save the trained weights (e.g. seq2squiggle_model.pt) and log the per-epoch combined loss to produce a loss_curve.png; at inference the duration sampler can be exchanged for static distributions via --duration-sampling False with --dwell-mean/--dwell-std.

## Related tools

- **seq2squiggle** (Deep-learning tool whose FFT-based architecture (encoder, length regulator, decoder, gamma duration predictor, Gaussian noise module) and combined MSE/NLL loss are being reconstructed and trained) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **uncalled4** (Generates the event-level signal-to-reference alignments (eventalign table) used as training chunks for the duration predictor) — https://github.com/skovaka/uncalled4
- **Python** (Implementation language (PyTorch) for the transformer, gamma duration sampler, and joint loss optimization)

## Examples

```
uncalled4 align input.ref input.slow5 --bam-in input.bam --eventalign-out events.tsv --eventalign-flags print-read-names,signal-index,samples --pore-model dna_r10.4.1_400bps_9mer --flowcell FLO-MIN114 --kit SQK-LSK114
```

## Evaluation signals

- The saved checkpoint seq2squiggle_model.pt loads without error in PyTorch
- loss_curve.png shows convergence of the combined training loss (signal MSE + duration NLL + noise MSE) on the uncalled4 eventalign chunks
- Each training chunk respects the interface: 16 k-mers segmented to at most 250 signal points
- Duration predictor outputs parameterize a valid gamma distribution (positive shape/scale) for per-k-mer signal lengths, and tau is set to 0.0005 in the NLL term
- At inference, toggling --duration-sampling False with --dwell-mean 10.0 --dwell-std 0.0 yields one signal sample per k-mer event (ideal event lengths)

## Limitations

- Testing workflow is not implemented in the repository, so correctness must be judged by checkpoint loading and loss-curve convergence rather than a test suite
- uncalled4 >= 4.1.0 no longer outputs scaled pA values directly, so an optional filter/transform step to pA values is needed before training
- The duration and noise samplers are stochastic modules; disabling them (--duration-sampling False, --noise-sampling False) falls back to static normal distributions, losing learned dwell-time variability

## Evidence

- [other] The mechanism is trained by summing three losses into a combined objective: signal MSE, duration NLL from a gamma-parameterizing duration sampler, and noise MSE from a Gaussian noise module, optimized jointly.: [evidence span withheld — anchor status: not_found]
- [other] Build the non-autoregressive transformer: a feed-forward transformer (FFT) encoder with 2 blocks, a length regulator, and an FFT decoder with 2 blocks.: [evidence span withheld — anchor status: not_found]
- [other] Load the uncalled4 eventalign training chunks, each covering 16 k-mers segmented to at most 250 signal points, and one-hot embed the nucleotide k-mer sequences.: [evidence span withheld — anchor status: not_found]
- [abstract] Unlike existing simulators relying on static k-mer models, seq2squiggle learns sequential contextual information from segmented signal data.: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"
- [readme] README: Duration Sampler predicts per-k-mer dwell time (event length) variation and can be disabled to use static distributions instead.: [evidence span withheld — anchor status: external_doc]
