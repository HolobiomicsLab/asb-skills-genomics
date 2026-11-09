---
name: cnn-rnn-architecture-ablation
description: Use when you need to demonstrate that both the convolutional and recurrent
  layer sets of Chiron's CNN+RNN+CTC architecture are essential, by training full
  vs. ablated models on the same raw-signal tfrecord data and comparing read-level
  accuracy against a reference genome.
when_to_use_negative:
- When the goal is only to basecall reads with a pretrained model (use the default
  DNA/RNA models via 'chiron call') rather than to test architecture components —
  no retraining is needed.
- When training labels have not been prepared (raw fast5 without re-squiggle/labeling
  via Metrichor + Nanoraw error correction) — the tfrecord extraction step requires
  signal-label pairs.
- When you only want to tune decoding (beam width vs. greedy decoding) — that is an
  inference-time choice and does not require retraining ablated architectures.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: Deep learning basecaller whose CNN+RNN+CTC model is trained in full and ablated
    forms; provides chiron train, chiron/utils/raw.py tfrecord extraction, and chiron
    call for basecalling evaluation reads
  repo: https://github.com/haotianteng/chiron
- name: h5py
  role: Python library used to read the .fast5 HDF5 files containing the raw nanopore
    signal during training-data extraction
  repo: https://github.com/h5py/h5py
- name: graphmap
  role: Aligner used with default parameters to align basecalled reads from each model
    variant to the reference genome for computing read-level accuracy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:caab94884f6531d4289dc2e5640bbfd4efbdbd8c31bc743db602680760581f54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Chiron
  - graphmap
  - h5py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cnn-rnn-architecture-ablation

## Summary

Ablation protocol for Chiron's CNN+RNN+CTC basecalling architecture: retrain the full model and two ablated variants (CNN-only and RNN-only removal) on the same ~4,000-read re-squiggled fast5 training set, then compare read-level accuracy to show each component performs a distinct, necessary step of the signal-to-sequence pipeline.

## When to use

Use when you need to demonstrate that both the convolutional and recurrent layer sets of Chiron's CNN+RNN+CTC architecture are essential, by training full vs. ablated models on the same raw-signal tfrecord data and comparing read-level accuracy against a reference genome.

## When NOT to use

- When the goal is only to basecall reads with a pretrained model (use the default DNA/RNA models via 'chiron call') rather than to test architecture components — no retraining is needed.
- When training labels have not been prepared (raw fast5 without re-squiggle/labeling via Metrichor + Nanoraw error correction) — the tfrecord extraction step requires signal-label pairs.
- When you only want to tune decoding (beam width vs. greedy decoding) — that is an inference-time choice and does not require retraining ablated architectures.

## Inputs

- Re-squiggled fast5 files of the ~4,000-read training set (raw signal + label pairs)
- Held-out evaluation fast5 reads for basecalling
- Reference genome for alignment (e.g. Phage Lambda NC_001416.1)
- GPU with ~8 GB GPU RAM, TensorFlow environment with Chiron installed

## Outputs

- Three trained models: full CNN+RNN+CTC, CNN-ablated, and RNN-ablated (distinct model_name entries in log_dir)
- Basecalled reads (fastq/fasta) from each of the three models
- Read-level accuracy values per model after graphmap alignment to the reference
- ablation_accuracy_comparison.csv with full-model vs. ablated-variant accuracies side by side

## How to apply

Extract raw signal and label pairs from the re-squiggled fast5 training files (~4,000 reads) using chiron/utils/raw.py in dna mode, which produces a tfrecord for training. Train the full CNN+RNN+CTC model with 'chiron train --data_dir <signal_label folder> --log_dir <model_log_folder> --model_name <saved_model_name>' on TensorFlow using a GPU (~8 GB GPU RAM recommended). Retrain two ablated variants — one with the convolutional layers removed and one with the recurrent layers removed — using the identical training command, data, and a distinct model_name in log_dir so the comparison isolates the architecture change. Basecall held-out evaluation reads with all three models, align the basecalled reads to the reference genome with graphmap (default parameters), and compute read-level accuracy per variant. Tabulate full-model vs. ablated accuracies (e.g. in ablation_accuracy_comparison.csv); the rationale is that the CNN extracts local patterns from the raw input signal while the RNN integrates those patterns into basecall probabilities, so removing either should eliminate a distinct pipeline step and degrade accuracy.

## Related tools

- **Chiron** (Deep learning basecaller whose CNN+RNN+CTC model is trained in full and ablated forms; provides chiron train, chiron/utils/raw.py tfrecord extraction, and chiron call for basecalling evaluation reads) — https://github.com/haotianteng/chiron
- **h5py** (Python library used to read the .fast5 HDF5 files containing the raw nanopore signal during training-data extraction) — https://github.com/h5py/h5py
- **graphmap** (Aligner used with default parameters to align basecalled reads from each model variant to the reference genome for computing read-level accuracy)

## Examples

```
python chiron/utils/raw.py --input <fast5 folder> --output <output_folder> --mode dna; chiron train --data_dir <signal_label folder> --log_dir <model_log_folder> --model_name <saved_model_name>
```

## Evaluation signals

- The full CNN+RNN+CTC model shows substantially higher read-level accuracy than both ablated variants, qualitatively reproducing the paper's reported accuracy drop when either component is removed.
- All three models are trained with the same command, same tfrecord data, and same GPU settings, differing only in the presence of the CNN or RNN layer set.
- Each ablated variant is saved under a distinct model_name in log_dir so results are attributable to the correct architecture.
- Read-level accuracy is computed after graphmap alignment to the reference genome with default parameters, matching the paper's evaluation protocol.
- The comparison is written to ablation_accuracy_comparison.csv with full-model and ablated accuracies tabulated side by side.

## Limitations

- The validation is qualitative — it reproduces the direction of the paper's accuracy drop, not an exact numeric match.
- Chiron is substantially more computationally expensive than Albacore and BasecRAWller, and training requires a GPU (~8 GB GPU RAM recommended).
- Training labels derive from Metrichor basecalling plus Nanoraw, so errors introduced by Metrichor must be corrected via alignment to avoid learning Metrichor's bias into Chiron.
- Batch size or window settings may need adjustment for limited RAM/GPU-RAM; increasing sliding window size improves accuracy at the cost of memory and running time.

## Evidence

- [intro] CNN and RNN roles: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder [13]"
- [discussion] training data preparation: "Metrichor, the basecaller provided by ONT that runs as a cloud service, is used to basecall the MinION sequencing data first. Then, Nanoraw [32] is used for labeling the data."
- [discussion] alignment protocol: "aligned to the reference genome using graphmap[33] with the default parameters"
- [readme] training command from README: "python chiron/entry.py call -i chiron/example_folder/ -o <output_folder> -m chiron/model/DNA_default --preset dna-pre"
- [readme] install and dependencies: "This will install Chiron, and [`h5py`](https://github.com/h5py/h5py) (required for reading in `.fast5` files)."
