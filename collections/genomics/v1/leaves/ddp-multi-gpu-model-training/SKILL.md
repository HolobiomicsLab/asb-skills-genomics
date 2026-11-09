---
name: ddp-multi-gpu-model-training
description: Use when you need to fine-tune or train a NanoSimFormer simulator model
  from an aligned BAM + POD5 training dataset, e.g. adapting the simulator to a new
  ONT chemistry (dna_r1041 or rna004).
when_to_use_negative:
- You only want to generate simulated signals with an already-trained model — use
  the simulate subcommand instead of retraining
- You need to build a training dataset rather than train — use create_dataset from
  aligned BAM + POD5 files first
- No CUDA GPU with compute capability >= 8.x is available, since DDP multi-GPU training
  requires Ampere/Ada/Hopper-class NVIDIA hardware
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NanoSimFormer
  role: Provides the train subcommand, dataset loader, BaseCaller subclass interface,
    and DDP training loop for the Transformer simulator
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: bonito
  role: Loads the official frozen ONT basecalling model (e.g. Dorado v5 super-accuracy
    preset) used to compute the basecaller-guided CTC loss
  repo: https://github.com/nanoporetech/bonito
- name: remora
  role: Performs signal-to-sequence alignment when preparing the training dataset
    (dwells.npy) used for the MSE loss targets
  repo: https://github.com/nanoporetech/remora
- name: PyTorch
  role: Provides torch.nn.functional.ctc_loss, DDP multi-GPU training, and the underlying
    deep-learning runtime
- name: flash-attn
  role: Attention kernel dependency installed via pre-built wheel (flash_attn-2.7.4.post1+cu126torch2.9-cp310)
    required for the Transformer model
  repo: https://github.com/mjun0812/flash-attention-prebuild-wheels
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:298e97bf16943c3ca8588c005c17fd093480c05caeb9edff5b7ecb252a4faa74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - NanoSimFormer
  - PyTorch
  - Python
  - bonito
  - flash-attn
  - remora
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ddp-multi-gpu-model-training

## Summary

Trains the NanoSimFormer Transformer signal simulator with Distributed Data Parallel (DDP) multi-GPU training, minimizing MSE losses on per-base mean signal level and noise standard deviation (targets from resquiggle-derived dwell times) plus a CTC NLL loss computed through a frozen Dorado v5 super-accuracy basecaller.

## When to use

Use when you need to fine-tune or train a NanoSimFormer simulator model from an aligned BAM + POD5 training dataset, e.g. adapting the simulator to a new ONT chemistry (dna_r1041 or rna004).

## When NOT to use

- You only want to generate simulated signals with an already-trained model — use the simulate subcommand instead of retraining
- You need to build a training dataset rather than train — use create_dataset from aligned BAM + POD5 files first
- No CUDA GPU with compute capability >= 8.x is available, since DDP multi-GPU training requires Ampere/Ada/Hopper-class NVIDIA hardware

## Inputs

- Training dataset files: signal.npy, sequence.npy, sequence_length.npy, dwells.npy (from create_dataset on aligned BAM + POD5)
- Pretrained ONT basecalling model checkpoint for preset dna_r1041 or rna004 (loaded via bonito)
- Model configuration file (--config) and chosen preset (--preset ont_r1041_dna_5khz or ont_rna004_4khz)

## Outputs

- Per-epoch model checkpoints written to weights/
- TensorBoard event logs in the output log/ directory
- Trained NanoSimFormer simulator weights usable for signal simulation via the simulate subcommand

## How to apply

Load the training dataset (signal.npy, sequence.npy, sequence_length.npy, dwells.npy) with the NanoSimFormer dataset loader and verify that per-base dwell times sum to the signal_chunk_len (5000 samples for dna_r1041, 4000 for rna004). Build the BaseCaller subclass (caller_name default ONTBasecaller) whose load_model loads the frozen pretrained ONT basecalling model via bonito for the chosen preset and puts it in eval mode so its weights are not updated. Compute the basecaller-guided CTC loss with torch.nn.functional.ctc_loss on log-softmax logits permuted to (time, batch, classes), encoding targets as A=1, C=2, G=3, T/U=4, pad=0 with unpadded lengths from sequence_length.npy. Add MSE terms on per-base signal mean and noise standard deviation computed against the resquiggle-derived dwell assignments in dwells.npy, then launch training with python -m nano_signal_simulator train under DDP multi-GPU (e.g. --ngpus 2), batch-size 64 (or 128), lr 0.0002, epochs 5, seed 42. Monitor loss curves with tensorboard --logdir on the output log/ directory and confirm per-epoch checkpoints are written to weights/.

## Related tools

- **NanoSimFormer** (Provides the train subcommand, dataset loader, BaseCaller subclass interface, and DDP training loop for the Transformer simulator) — https://github.com/BioinfoSZU/NanoSimFormer
- **bonito** (Loads the official frozen ONT basecalling model (e.g. Dorado v5 super-accuracy preset) used to compute the basecaller-guided CTC loss) — https://github.com/nanoporetech/bonito
- **remora** (Performs signal-to-sequence alignment when preparing the training dataset (dwells.npy) used for the MSE loss targets) — https://github.com/nanoporetech/remora
- **PyTorch** (Provides torch.nn.functional.ctc_loss, DDP multi-GPU training, and the underlying deep-learning runtime)
- **flash-attn** (Attention kernel dependency installed via pre-built wheel (flash_attn-2.7.4.post1+cu126torch2.9-cp310) required for the Transformer model) — https://github.com/mjun0812/flash-attention-prebuild-wheels

## Examples

```
python -m nano_signal_simulator train --ngpus 2 --batch-size 64 --lr 0.0002 --epochs 5 --seed 42; tensorboard --logdir log/
```

## Evaluation signals

- Per-base dwell times in dwells.npy sum exactly to signal_chunk_len (5000 for dna_r1041, 4000 for rna004) before training starts
- Frozen basecaller weights remain unchanged after training (model kept in eval mode; only the simulator Transformer updates)
- TensorBoard loss curves in log/ show combined CTC NLL + two MSE terms decreasing across the 5 epochs without divergence at lr 0.0002
- A per-epoch checkpoint file appears in weights/ after each epoch completes under DDP with --ngpus 2
- Trained model, when used with simulate --basecall, yields simulated reads whose median basecalling accuracy exceeds 99% (paper benchmark)

## Limitations

- Requires Linux (fully tested on Ubuntu 22.04) and an NVIDIA GPU with CUDA compute capability >= 8.x with driver >= 525
- For pip/conda installs, pretrained model checkpoints must be manually downloaded from Zenodo (10.5281/zenodo.20364515) and unzipped into the package install path; the Docker image ships them pre-installed
- Training recipes are chemistry-specific: signal_chunk_len and basecaller preset must match the target chemistry (dna_r1041 vs rna004); adapting to future ONT chemistries requires the documented extension workflow

## Evidence

- [full_text] Training minimizes MSE losses on per-base mean signal level and noise standard deviation plus a CTC negative log-likelihood loss computed through a pre-trained, frozen basecaller: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract basecaller guidance quote: "an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals"
- [methods] Methods train subcommand quote: "train             Train a NanoSimFormer model"
- [readme] README install quote: "pip install nanosimformer -i https://pypi.org/simple --extra-index-url https://download.pytorch.org/whl/cu126"
- [readme] README hardware quote: "NanoSimFormer is compatible with Linux and has been fully tested on Ubuntu 22.04."
