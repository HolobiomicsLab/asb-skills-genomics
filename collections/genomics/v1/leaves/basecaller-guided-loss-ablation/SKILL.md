---
name: basecaller-guided-loss-ablation
description: Use when you need to attribute NanoSimFormer's high signal fidelity to
  its basecaller-guided CTC training objective by retraining the identical architecture
  without it and comparing median read basecalling accuracy.
when_to_use_negative:
- When the goal is simulation performance benchmarking with the published full model
  — use the pre-trained basecaller-guided checkpoints from Zenodo instead of retraining.
- When testing changes to architecture (e.g. attention layout) rather than the training
  objective — this ablation isolates only the loss term.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSimFormer
  role: Transformer signal simulator whose train subcommand is modified to disable
    the CTC basecalling loss term
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated reads (--basecall) to compute median read accuracy of
    the ablated checkpoint
  repo: https://github.com/nanoporetech/dorado
- name: flash-attn
  role: Flash-attention dependency required in the Python 3.10 training environment
  repo: https://github.com/mjun0812/flash-attention-prebuild-wheels
- name: Docker
  role: Pre-built chobits323/nano-sim:v1.1 image provides the recommended training
    environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:317f51f5927e44b90da4e5aa03243362a5926821c50c0e111988041064a6cf95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Python
  - conda
  - flash-attn
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# basecaller-guided-loss-ablation

## Summary

Ablation protocol that disables the basecaller-guided CTC loss term in NanoSimFormer's Transformer generator while keeping event-level mean and noise losses, to quantify whether basecaller guidance (rather than architecture) drives signal fidelity.

## When to use

Use when you need to attribute NanoSimFormer's high signal fidelity to its basecaller-guided CTC training objective by retraining the identical architecture without it and comparing median read basecalling accuracy.

## When NOT to use

- When the goal is simulation performance benchmarking with the published full model — use the pre-trained basecaller-guided checkpoints from Zenodo instead of retraining.
- When testing changes to architecture (e.g. attention layout) rather than the training objective — this ablation isolates only the loss term.

## Inputs

- HG002 R10.4.1 training dataset from create_dataset (signal.npy, sequence.npy, sequence_length.npy, dwells.npy)
- Aligned BAM + POD5 source files for dataset creation
- NanoSimFormer source code (nano_signal_simulator package)

## Outputs

- Ablated model checkpoints (epoch_N.checkpoint.pth / step_M.checkpoint.pth under training_output/weights)
- TensorBoard training logs (training_output/log, events.out.* files)
- Median read basecalling accuracy of the ablated model from Dorado-based evaluation

## How to apply

Start from an HG002 R10.4.1 training dataset produced by `create_dataset` (signal.npy, sequence.npy, sequence_length.npy, dwells.npy) in a Python 3.10 environment (nano-sim Docker image chobits323/nano-sim:v1.1 or conda env with flash-attn). Modify the training objective in `nano_signal_simulator train` to skip the ONTBasecaller CTC basecalling_loss term while keeping the identical Transformer generator and dataset. Launch multi-GPU DDP training with the dna_r1041 preset (--batch-size 128, --epochs 5, --lr 0.0002, --seed 42, --ngpus 2, --save_every_steps 5000), monitoring convergence via TensorBoard under training_output/log and checkpoints under training_output/weights. Evaluate the ablated checkpoint by simulating reads (--preset ont_r1041_dna_5khz) and basecalling them with Dorado (--basecall) to compute median read basecalling accuracy.

## Related tools

- **NanoSimFormer** (Transformer signal simulator whose train subcommand is modified to disable the CTC basecalling loss term) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated reads (--basecall) to compute median read accuracy of the ablated checkpoint) — https://github.com/nanoporetech/dorado
- **flash-attn** (Flash-attention dependency required in the Python 3.10 training environment) — https://github.com/mjun0812/flash-attention-prebuild-wheels
- **Docker** (Pre-built chobits323/nano-sim:v1.1 image provides the recommended training environment)

## Examples

```
python -m nano_signal_simulator train --preset dna_r1041 --batch-size 128 --epochs 5 --lr 0.0002 --seed 42 --ngpus 2 --save_every_steps 5000
```

## Evaluation signals

- Ablated model's median read basecalling accuracy falls within the reported ablation range of 93.18–97.69%, comparable to seq2squiggle
- Full basecaller-guided model exceeds 99% median basecalling accuracy, confirming the gap is attributable to the loss term
- Checkpoints appear as epoch_N.checkpoint.pth / step_M.checkpoint.pth under training_output/weights with --save_every_steps 5000
- TensorBoard events.out.* files in training_output/log show training convergence across the 5 epochs

## Limitations

- Ablation requires retraining from scratch (5 epochs, multi-GPU DDP); it cannot be inferred from pre-trained checkpoints alone.
- Requires NVIDIA GPU with CUDA compute capability >= 8.x (Ampere or newer) and NVIDIA driver >= 525; Linux-tested (Ubuntu 22.04).
- The conclusion is specific to the HG002 R10.4.1 DNA dataset and the ONT R10.4.1 preset; other chemistries may require dataset regeneration.

## Evidence

- [results] Without basecaller guidance, the ablated model degrades to median read accuracies of 93.18–97.69%, comparable to seq2squiggle, demonstrating that basecaller-guided training is the primary driver of signal fidelity.: [evidence span withheld — anchor status: not_found]
- [methods] The --basecall option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [abstract] an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals: "an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals"
- [readme] GPU: NVIDIA GPU with CUDA compute capability >= 8.x (e.g., Ampere, Ada, or Hopper GPUs like A100, RTX 3090, RTX 4090, H100): [evidence span withheld — anchor status: not_found]
- [readme] create_dataset    Create a training dataset from aligned BAM + POD5 files: "create_dataset    Create a training dataset from aligned BAM + POD5 files"
