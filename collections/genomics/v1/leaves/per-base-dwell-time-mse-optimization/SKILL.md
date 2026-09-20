---
name: per-base-dwell-time-mse-optimization
description: Use when training or fine-tuning a nanopore signal simulator whose per-base
  signal levels must match resquiggle-derived dwell-time statistics for ONT DNA R10.4.1
  or direct-RNA RNA004 chemistry.
when_to_use_negative:
- You only need read-level simulation from a reference without training a simulator
  — use the simulate subcommand instead of the train workflow.
- No aligned BAM + POD5 training data with resquiggle (Remora) signal-to-sequence
  alignment is available to produce dwells.npy targets.
- Your sequencing chemistry is not dna_r1041 or rna004 — adapting requires the paper's
  extensibility workflow rather than the default presets.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: NanoSimFormer
  role: End-to-end Transformer signal simulator providing create_dataset and train
    subcommands that implement the per-base dwell-time MSE training objective
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: remora
  role: Performs signal-to-sequence alignment during training dataset preparation
    to derive per-base dwell times used as MSE targets
  repo: https://github.com/nanoporetech/remora
- name: bonito
  role: Loads the frozen pretrained ONT basecalling model (Dorado v5 super-accuracy)
    whose CTC loss is combined with the dwell-time MSE losses
  repo: https://github.com/nanoporetech/bonito
- name: Dorado
  role: Supplies the super-accuracy basecaller used as frozen guidance during simulator
    training
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:398392b504d4372bde99b50f48434d1d0c7c4cc127286027498d6b7030fd870b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Dorado
  - NanoSimFormer
  - PyTorch
  - Python
  - bonito
  - remora
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# per-base-dwell-time-mse-optimization

## Summary

Train the NanoSimFormer Transformer signal simulator with MSE losses on per-base mean signal level and noise standard deviation, where the targets derive from resquiggle-based per-base dwell times (dwells.npy) computed during dataset creation with Remora signal-to-sequence alignment.

## When to use

Use when training or fine-tuning a nanopore signal simulator whose per-base signal levels must match resquiggle-derived dwell-time statistics for ONT DNA R10.4.1 or direct-RNA RNA004 chemistry.

## When NOT to use

- You only need read-level simulation from a reference without training a simulator — use the simulate subcommand instead of the train workflow.
- No aligned BAM + POD5 training data with resquiggle (Remora) signal-to-sequence alignment is available to produce dwells.npy targets.
- Your sequencing chemistry is not dna_r1041 or rna004 — adapting requires the paper's extensibility workflow rather than the default presets.

## Inputs

- Aligned BAM + POD5 files from ONT DNA R10.4.1 or direct-RNA RNA004 reads
- Training dataset chunks: signal.npy, sequence.npy, sequence_length.npy, dwells.npy (per-base dwell times summing to 5000 samples for dna_r1041 or 4000 for rna004)
- Frozen pretrained ONT basecalling model checkpoint (Dorado v5 super-accuracy, loaded via bonito; Zenodo doi 10.5281/zenodo.20364515 for pip installs)

## Outputs

- Trained NanoSimFormer Transformer simulator checkpoints written per-epoch to weights/
- TensorBoard training-loss logs in the output log/ directory
- Simulator generating synthetic POD5 nanopore signals whose per-base mean levels and noise standard deviations match resquiggle-derived dwell statistics

## How to apply

First create a training dataset from aligned BAM + POD5 files using NanoSimFormer's create_dataset, where Remora performs signal-to-sequence alignment and each chunk stores signal.npy, sequence.npy, sequence_length.npy, and dwells.npy; verify per-base dwell times sum to signal_chunk_len (5000 samples for dna_r1041, 4000 for rna004). Apply the dwell-based filters (--min_dwell 4, --max_dwell 256 signal samples, plus --min_identity 0.99 and --min_align_ratio 0.995) to exclude unreliable per-base level assignments. During training, compute MSE losses on the per-base mean signal level and the per-base noise standard deviation against targets derived from the resquiggle dwell assignments, and combine them with a CTC negative log-likelihood loss computed through a frozen Dorado v5 super-accuracy basecaller loaded via bonito (ont-koi forward-backward over batches of simulated signals). Launch training with python -m nano_signal_simulator train using DDP multi-GPU (e.g. --ngpus 2), batch-size 64 or 128, lr 0.0002, 5 epochs, seed 42, and monitor the loss in the output log/ directory with TensorBoard while checkpoints are written to weights/. The rationale is that level-and-noise supervision anchors simulated signal statistics to real measured dwell distributions, while basecaller guidance keeps simulated signals decodable.

## Related tools

- **NanoSimFormer** (End-to-end Transformer signal simulator providing create_dataset and train subcommands that implement the per-base dwell-time MSE training objective) — https://github.com/BioinfoSZU/NanoSimFormer
- **remora** (Performs signal-to-sequence alignment during training dataset preparation to derive per-base dwell times used as MSE targets) — https://github.com/nanoporetech/remora
- **bonito** (Loads the frozen pretrained ONT basecalling model (Dorado v5 super-accuracy) whose CTC loss is combined with the dwell-time MSE losses) — https://github.com/nanoporetech/bonito
- **Dorado** (Supplies the super-accuracy basecaller used as frozen guidance during simulator training) — https://github.com/nanoporetech/dorado

## Examples

```
python -m nano_signal_simulator train --preset ont_r1041_dna_5khz --ngpus 2 --batch-size 64 --lr 0.0002 --epochs 5 --seed 42
```

## Evaluation signals

- Per-base dwell times in dwells.npy sum exactly to signal_chunk_len: 5000 samples for dna_r1041 or 4000 samples for rna004.
- Dwell-time filter invariants hold: every per-base dwell lies between --min_dwell 4 and --max_dwell 256 signal samples.
- Frozen basecaller weights are unchanged after training: the bonito-loaded model is in eval mode and no gradients update it.
- Training loss curves in the TensorBoard log/ directory decrease across the 5 epochs (lr 0.0002, batch 64/128, seed 42) and per-epoch checkpoints appear in weights/.
- Trained simulator reaches median basecalling accuracy exceeding 99% and Q-scores above 22.8 on ONT R10.4.1 DNA and direct RNA data.

## Limitations

- The skill is chemisty-specific: default presets cover only dna_r1041 (5 kHz, 5000-sample chunks) and rna004 (4 kHz, 4000-sample chunks); other chemistries require the paper's adaptation workflow.
- High-fidelity per-base dwell targets depend on quality filtering (default min_identity 0.99, min_quality 20.0, min_align_ratio 0.995, min_aligned_read_len 1000, MAPQ 1); noisy or misaligned reads degrade dwell statistics.
- Pre-trained checkpoints must be downloaded from Zenodo (10.5281/zenodo.20364515) for pip/conda installs; the Docker image chobits323/nano-sim:v1.1 already bundles them.

## Evidence

- [methods] Training minimizes MSE losses on per-base mean signal level and noise standard deviation: [evidence span withheld — anchor status: not_found]
- [methods] remora for signal-to-sequence alignment: "remora (https://github.com/nanoporetech/remora) for signal-to-sequence alignment during training dataset preparation"
- [methods] create_dataset builds training data: "create_dataset    Create a training dataset from aligned BAM + POD5 files"
- [other] verify dwell sums: [evidence span withheld — anchor status: not_found]
- [readme] filter thresholds from README: "--min_dwell MIN_DWELL Minimum per-base dwell time (signal samples). (default: 4)  --max_dwell MAX_DWELL Maximum per-base dwell time (signal samples). (default: 256)"
