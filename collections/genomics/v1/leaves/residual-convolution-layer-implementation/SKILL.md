---
name: residual-convolution-layer-implementation
description: Use when reconstructing or re-implementing Chiron's segmentation-free
  basecalling network in TensorFlow, specifically when building the CNN portion that
  converts raw .fast5 electrical signal into features for the bidirectional LSTM layers.
when_to_use_negative:
- The basecalling pipeline already exists (e.g. pip-installed chiron) and you only
  need to run inference with `chiron call` rather than rebuild the architecture
- You need the recurrent or decoding portion of the network (bidirectional LSTM stacking,
  softmax, CTC greedy/beam decoders) rather than the convolutional feature extractor
- Input is already-segmented k-mer signal events from the older two-stage basecalling
  approach; Chiron's residual CNN operates directly on raw, unsegmented signal
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: Reference implementation whose CNN+RNN+CTC architecture is being reconstructed;
    provides model_definition.py and the DNA_default model
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: Framework used to build the computational graph of the residual convolutional
    layers (tensorflow==1.15, Python 2.7)
- name: h5py
  role: Reads .fast5 files containing the raw nanopore electrical signal fed to the
    conv layers
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:c084dbfed4ab7f89c49ec40419a25a6f52cf0faa2cbc744cc08fd3ee902ccdae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# residual-convolution-layer-implementation

## Summary

Implement the convolutional feature extractor of Chiron's CNN+RNN+CTC basecaller: five residual convolutional layers (256 channels, ReLU, batch normalization) that discriminate local patterns in raw nanopore electrical signal, feeding per-timestep features into the downstream recurrent/CTC stack.

## When to use

Use when reconstructing or re-implementing Chiron's segmentation-free basecalling network in TensorFlow, specifically when building the CNN portion that converts raw .fast5 electrical signal into features for the bidirectional LSTM layers.

## When NOT to use

- The basecalling pipeline already exists (e.g. pip-installed chiron) and you only need to run inference with `chiron call` rather than rebuild the architecture
- You need the recurrent or decoding portion of the network (bidirectional LSTM stacking, softmax, CTC greedy/beam decoders) rather than the convolutional feature extractor
- Input is already-segmented k-mer signal events from the older two-stage basecalling approach; Chiron's residual CNN operates directly on raw, unsegmented signal

## Inputs

- Raw nanopore electrical signal from .fast5 files (read via h5py)
- Model definition parameters (256 channels, 5 residual conv layers, preset dna-pre)

## Outputs

- model_definition.py containing the convolutional feature extractor within the CNN+RNN+CTC computational graph
- Per-timestep convolved signal features passed to the bidirectional LSTM layers

## How to apply

In a Python 2.7 environment with TensorFlow 1.15, define five residual convolutional layers with 256 channels, ReLU activations, and batch normalization, mirroring Chiron's CNN+RNN+CTC structure in model_definition.py. Each residual block operates directly on the raw input signal (no event segmentation step), producing local-pattern features that are consumed by three bidirectional LSTM layers with 200 hidden units, a fully-connected layer, and a softmax over the nucleotide alphabet. The convolved features are fed per-timestep to the RNN, so keep tensor shapes consistent with the CTC loss/decoder requirements. Validate by constructing the full computational graph and confirming the CNN, RNN, softmax, and CTC/decoder components are all present and the script runs without error.

## Related tools

- **Chiron** (Reference implementation whose CNN+RNN+CTC architecture is being reconstructed; provides model_definition.py and the DNA_default model) — https://github.com/haotianteng/Chiron
- **TensorFlow** (Framework used to build the computational graph of the residual convolutional layers (tensorflow==1.15, Python 2.7))
- **h5py** (Reads .fast5 files containing the raw nanopore electrical signal fed to the conv layers) — https://github.com/h5py/h5py

## Examples

```
pip install tensorflow==1.15; python chiron/entry.py call -i chiron/example_folder/ -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- model_definition.py exists and contains the CNN, RNN, softmax, and CTC/decoder components
- The definition script runs and constructs the TensorFlow 1.15 computational graph without error
- Convolutional stack consists of five residual layers with 256 channels, ReLU, and batch normalization per the paper's description
- Downstream bidirectional LSTM layers (3 x 200 hidden units) accept the convolved per-timestep features without shape errors
- Test basecalling on the 5 sample DNA .fast5 files in chiron/example_data/ with the DNA_default model produces fastq/fasta output in the result folder

## Limitations

- Chiron is substantially more computationally expensive than Albacore and somewhat more than BasecRAWller; the conv stack's inference cost scales with sliding-window size
- Built with Python 2.7 and TensorFlow 1.15; version clashes require a dedicated virtual environment
- Increasing the sliding-window size improves accuracy but at the cost of increased memory and running time

## Evidence

- [other] Chiron's described segmentation-free network consists of convolutional layers that discriminate local patterns in the raw input signal, recurrent layers that integrate these patterns into basecall probabilities, and a CTC decoder on top: [evidence span withheld — anchor status: not_found]
- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [readme] README: Built with TensorFlow and python 2.7; install tensorflow==1.15 alongside pip-installed chiron: "Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer.  
Built with **TensorFlow** and python 2.7."
- [readme] README: test-run example uses the 5 sample DNA fast5 files and DNA_default model with dna-pre preset: "python chiron/entry.py call -i chiron/example_folder/ -o <output_folder> -m chiron/model/DNA_default --preset dna-pre"
