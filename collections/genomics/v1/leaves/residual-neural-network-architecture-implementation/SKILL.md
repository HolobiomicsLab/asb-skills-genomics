---
name: residual-neural-network-architecture-implementation
description: Use when implementing or reproducing a deep-learning m6A detector on
  Nanopore DRS reads that combines a 65-sample rectified signal window with 5-mer
  basecalling-error features (quality, mismatch, insertion, deletion) into per-site
  modification probabilities.
when_to_use_negative:
- Input is already a per-site probability table — the architecture operates on read-level
  feature rows, not aggregated prediction outputs
- Data is not Nanopore direct RNA sequencing-derived signal/error features (e.g. Illumina
  or m6A-seq peak data), which lack the 65-sample rectified signal window and basecalling-error
  columns RedNano expects
- Sites have fewer than 20 supporting reads, below the multiple-instance-learning
  coverage threshold
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3524
tools:
- name: RedNano
  role: Source implementation of the dual-ResNet architecture, MIL aggregation, and
    train/predict scripts
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch
  role: Framework for implementing the two residual networks, fully connected fusion
    head, and sigmoid output
- name: CUDA
  role: GPU backend used to train and run prediction (CUDA_VISIBLE_DEVICES=0, CUDA
    11.1)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:183ad2eadf531b13bd3d36903df15b8a4d501a12137a86dd95933b11cf14af55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch
  - Python
  - RedNano
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# residual-neural-network-architecture-implementation

## Summary

Reconstruct the RedNano dual-ResNet deep-learning architecture that encodes Nanopore direct RNA sequencing raw current signals and basecalling errors with two parallel PyTorch residual networks, then fuses them via a fully connected head with a sigmoid output for read-level m6A probability and multiple-instance-learning aggregation to site level.

## When to use

Use when implementing or reproducing a deep-learning m6A detector on Nanopore DRS reads that combines a 65-sample rectified signal window with 5-mer basecalling-error features (quality, mismatch, insertion, deletion) into per-site modification probabilities.

## When NOT to use

- Input is already a per-site probability table — the architecture operates on read-level feature rows, not aggregated prediction outputs
- Data is not Nanopore direct RNA sequencing-derived signal/error features (e.g. Illumina or m6A-seq peak data), which lack the 65-sample rectified signal window and basecalling-error columns RedNano expects
- Sites have fewer than 20 supporting reads, below the multiple-instance-learning coverage threshold

## Inputs

- Sorted, site-aggregated RedNano feature TSV (columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele) filtered to coverage >= 20
- Train/valid labeled feature files with appended 0/1 label column for training
- Pre-trained checkpoint (e.g. models/mil_allspecies_model_states.pt)

## Outputs

- Trained PyTorch checkpoint saved to train/
- Per-site results TSV with columns transcript, pos, strand, prob, label, where prob is the modification probability in [0, 1]

## How to apply

In PyTorch, define two parallel input stems (1D convolution plus max-pooling): one embedding the 5-mer sequence channel (embedding_size 4, seq_lens 5) and one consuming the 65-sample rectified signal window (signal_lens 65). Feed each stem into its own residual network of four basic blocks with 32/64/128/256 channels, where each block contains four convolutional layers with identical output channel numbers and kernel size, encoding the basecalling-error and raw-signal branches separately (model_type comb_basecall_raw). Concatenate the two branch representations and pass them through a two-hidden-layer fully connected network (hidden_size 128, dropout_rate 0.5) ending in a sigmoid unit that emits the read-level modification probability. Aggregate read-level predictions into a site-level probability per transcript position using the multiple-instance-learning approach on features grouped by site with coverage >= 20 reads. Train with scripts/train.py (batch_size 4, lr 1e-4, weight_decay 1e-5, clip_grad 0.5, 50 epochs with patience 5 early stopping) and predict with scripts/predict.py (num_iterations 1000, batch_size 1, nproc 3).

## Related tools

- **RedNano** (Source implementation of the dual-ResNet architecture, MIL aggregation, and train/predict scripts) — https://github.com/Derryxu/RedNano
- **PyTorch** (Framework for implementing the two residual networks, fully connected fusion head, and sigmoid output)
- **CUDA** (GPU backend used to train and run prediction (CUDA_VISIBLE_DEVICES=0, CUDA 11.1))

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw --output_file test.features.sort.aggre.c20.results.tsv --nproc 3
```

## Evaluation signals

- The training run produces a checkpoint file in the --save_dir (train/) directory
- scripts/predict.py executes end-to-end with --model pointing at the checkpoint and emits a results TSV
- Every modification probability in the output table falls in [0, 1], consistent with the sigmoid output unit
- Output table schema matches transcript, pos, strand, prob, label with one row per transcript position passing the coverage >= 20 filter
- Reconstruction uses the exact hyperparameters: embedding_size 4, seq_lens 5, signal_lens 65, hidden_size 128, dropout_rate 0.5, model_type comb_basecall_raw

## Limitations

- Architecture hyperparameters (65-sample signal window, 5-mer k-mer, four residual blocks with 32/64/128/256 channels) are specific to RedNano's comb_basecall_raw model type and would need redesign for other k-mer or window lengths
- Site-level aggregation requires at least 20 reads per site for the MIL approach, so low-coverage transcript positions are dropped
- Pre-trained mil_allspecies_model_states.pt is a DRACH m6A model trained on synthetic, human, and Arabidopsis data; performance on other modifications or species relies on retraining
- Training on private data requires manually adding a 0/1 label column and splitting modified/unmodified feature files

## Evidence

- [methods] RedNano processes the two feature types with two separate ResNets, each comprising four basic residual blocks containing four convolutional layers with identical output channel numbers and kernel size.: [evidence span withheld — anchor status: not_found]
- [intro] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads.: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads."
- [readme] CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw: "CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65"
- [readme] CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5: "CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5"
