---
name: ctc-neural-network-training
description: Use when raw nanopore fast5 signals and reference-derived labels are
  available and a custom basecalling model must be trained or fine-tuned (e.g. new
  species, modified bases) instead of using a pretrained basecaller.
when_to_use_negative:
- You only need to basecall with an existing model — use 'chiron call' with the DNA_default/RNA_default
  pretrained models instead of training.
- Input data lacks ground-truth labels (reference genome or reliable basecalled labels);
  CTC training requires signal-label pairs from re-squiggled fast5 files.
- The target is non-nanopore signal data or the fast5 files have not been re-squiggled
  (e.g. via Tombo), so raw signal and labels cannot be paired.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: Chiron
  role: the CNN+RNN+CTC basecaller whose training entry point (chiron train) is executed;
    installed via pip or git clone
  repo: https://github.com/haotianteng/chiron
- name: TensorFlow
  role: deep learning framework (v1.15, GPU build with CUDA/cuDNN) in which the CTC-trained
    model is built and trained
- name: h5py
  role: reads the .fast5 files containing raw signal during export of training pairs
  repo: https://github.com/h5py/h5py
- name: graphmap
  role: aligns basecalled reads to the reference to compute read-level accuracy of
    the trained checkpoint
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:60ad70dcfa331b7f003be8b9fa5bf90b0f6c4e1044d52b7e6205f06ea01a155d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Chiron
  - TensorFlow
  - graphmap
  - h5py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ctc-neural-network-training

## Summary

Train a CNN+RNN+CTC deep neural network (Chiron) on raw nanopore signal-to-label pairs to perform end-to-end basecalling, using TensorFlow with CTC loss so the network learns to translate raw electrical signal directly into nucleotide sequence without an event segmentation step.

## When to use

Use when raw nanopore fast5 signals and reference-derived labels are available and a custom basecalling model must be trained or fine-tuned (e.g. new species, modified bases) instead of using a pretrained basecaller.

## When NOT to use

- You only need to basecall with an existing model — use 'chiron call' with the DNA_default/RNA_default pretrained models instead of training.
- Input data lacks ground-truth labels (reference genome or reliable basecalled labels); CTC training requires signal-label pairs from re-squiggled fast5 files.
- The target is non-nanopore signal data or the fast5 files have not been re-squiggled (e.g. via Tombo), so raw signal and labels cannot be paired.

## Inputs

- re-squiggled fast5 files of ~4,000 reads (e.g. 2,000 E. coli + 2,000 Lambda, ~4 Mbp) from a MinION R9.4 run
- signal-label tfrecord file produced by chiron/utils/raw.py or 'chiron export' (labels from Metrichor + Nanoraw with alignment-based error correction)
- GPU machine with ≥8 GB GPU RAM, CUDA and cuDNN installed, TensorFlow 1.15

## Outputs

- model checkpoints written to log_dir/model_name/
- a trained Chiron model usable via 'chiron call' for DNA or RNA basecalling
- read-level accuracy metrics on a held-out cross-validation remainder

## How to apply

First export signal-label pairs from re-squiggled fast5 files using chiron/utils/raw.py or 'chiron export', producing a tfrecord training file; labels come from Metrichor basecalling plus Nanoraw error correction so Metrichor's own errors are corrected against the reference before training. Configure a GPU environment with TensorFlow 1.15 (tensorflow-gpu==1.15), CUDA and cuDNN, with at least ~8 GB GPU RAM (the paper reports 3 epochs over 4,000 E. coli + Lambda reads took ~10 h on an Nvidia K80). Launch 'chiron train --data_dir <signal_label folder> --log_dir <model_log_folder> --model_name <name>', choosing batch_size and sequence_len to fit GPU memory and setting step_rate (learning rate) and max_step for the desired epoch count; checkpoints accumulate in log_dir/model_name/. Monitor the training log for convergence, then evaluate the checkpoint's read-level accuracy on a cross-validation remainder (reads excluded from training) after aligning basecalled reads to the reference with graphmap. The CNN layers extract local signal patterns while the recurrent layers integrate them into basecall probabilities, so the full CNN+RNN+CTC architecture is required — ablations of either component substantially degrade accuracy.

## Related tools

- **Chiron** (the CNN+RNN+CTC basecaller whose training entry point (chiron train) is executed; installed via pip or git clone) — https://github.com/haotianteng/chiron
- **TensorFlow** (deep learning framework (v1.15, GPU build with CUDA/cuDNN) in which the CTC-trained model is built and trained)
- **h5py** (reads the .fast5 files containing raw signal during export of training pairs) — https://github.com/h5py/h5py
- **graphmap** (aligns basecalled reads to the reference to compute read-level accuracy of the trained checkpoint)

## Examples

```
chiron train --data_dir <signal_label folder> --log_dir <model_log_folder> --model_name <saved_model_name>
```

## Evaluation signals

- A model checkpoint exists in log_dir/model_name/ after training completes.
- Wall-clock training time is consistent with the reported ~10 hours for 3 epochs over 4,000 reads on a K80-class GPU.
- Read-level accuracy on the cross-validation remainder is consistent with state-of-the-art basecalling quality reported for a model trained on only 4,000 reads.
- Ablation check: removing either the CNN or RNN layers and retraining yields substantially lower read-level accuracy than the full model.
- Alignment rate of basecalled reads to the reference (via graphmap, default parameters) is high enough to score accuracy per read.

## Limitations

- Chiron is substantially more computationally expensive than Albacore and BasecRAWller.
- Training labels derive from Metrichor, whose errors must be corrected via alignment to avoid learning Metrichor bias into the model.
- GPU memory limits batch_size and sequence_len; the sliding window trades accuracy against memory and running time.
- Accuracy gap of up to 2% vs Albacore remains on Human DNA; broader training data is expected to improve this.

## Evidence

- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [results] Training took 10 hours on K80: [evidence span withheld — anchor status: not_found]
- [discussion] Metrichor labels corrected via alignment: "from the alignment, the errors introduced by Metrichor are corrected to avoid the bias from Metrichor being learned into Chiron."
- [readme] pip install tensorflow-gpu==1.15: "pip install tensorflow-gpu==1.15"
- [readme] chiron train command writes checkpoints: [evidence span withheld — anchor status: not_found]
