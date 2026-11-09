---
name: multitask-convolutional-network-implementation
description: Use when reconstructing or reimplementing Clairvoyante's five-layer CNN
  architecture for variant calling from SMS reads, or when building a multitask DNN
  that predicts variant type, zygosity, alternative allele and indel length from alignment
  tensors.
when_to_use_negative:
- Variants with two or more alternative alleles — the current Clairvoyante design
  ignores multi-allelic variants and multi-allelic SNPs are excluded from training
- Indels longer than 4 bp when an exact alternative allele is needed — due to rareness
  of long indel training samples, the exact alternative allele is only provided for
  indels ≤4 bp
- Somatic or trio-sample-based variant calling — not supported by this implementation
  (planned future work)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Tensorflow
  role: Machine-learning framework on which the five-layer Clairvoyante network is
    built and trained
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Clairvoyante
  role: The multi-task five-layer CNN variant caller itself; open-source with modules
    to train, utilize and visualize the model
  repo: https://github.com/aquaskyline/Clairvoyante
- name: HKU-BAL/Clairvoyante-PyTroch
  role: PyTorch reimplementation with the exact same APIs as the TensorFlow clairvoyante_v3.py
    model, using NCHW tensor permutation
  repo: https://github.com/HKU-BAL/Clairvoyante-PyTroch
- name: Blosc/python-blosc
  role: Python dependency of the Clairvoyante toolset used for tensor compression
    in the data-preparation/calling pipeline
  repo: https://github.com/Blosc/python-blosc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:1176d3bde8566b13595bd5925275a73582954daf90c19a40fed0b1a235ea322e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Blosc/python-blosc
  - Clairvoyante
  - HKU-BAL/Clairvoyante-PyTroch
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multitask-convolutional-network-implementation

## Summary

Implement Clairvoyante, a multi-task five-layer convolutional neural network (three conv layers plus two fully connected layers with four prediction heads) for variant calling on single-molecule sequencing reads, built on TensorFlow. The network jointly predicts variant type (SNP or indel), zygosity, alternative allele, and indel length from encoded read-alignment tensors.

## When to use

Use when reconstructing or reimplementing Clairvoyante's five-layer CNN architecture for variant calling from SMS reads, or when building a multitask DNN that predicts variant type, zygosity, alternative allele and indel length from alignment tensors.

## When NOT to use

- Variants with two or more alternative alleles — the current Clairvoyante design ignores multi-allelic variants and multi-allelic SNPs are excluded from training
- Indels longer than 4 bp when an exact alternative allele is needed — due to rareness of long indel training samples, the exact alternative allele is only provided for indels ≤4 bp
- Somatic or trio-sample-based variant calling — not supported by this implementation (planned future work)

## Inputs

- 33x4x4 input tensors per candidate variant locus (reference base, insertion count, deletion count, alternate-allele count channels over a 33 bp window with 16-bp flanks)
- Training tensors derived from Genome-in-a-Bottle (GIAB) version 3.3.2 for HG001
- Truth variant sites paired 1:2 with non-variants randomly sampled from the genome

## Outputs

- Trained TensorFlow model checkpoint (~1.63M parameters)
- Four prediction heads: alternative allele, zygosity, variant type (SNP or indel), and indel length
- Training report with parameter count, output-head shapes, and final validation loss

## How to apply

Encode each candidate variant locus as a 33x4x4 input tensor covering a 33 bp window (candidate position plus 16-bp flanks on each side) with four channels: reference base, insertion count, deletion count, and alternate-allele count from the read alignment. Assemble the five-layer network in TensorFlow: three convolutional layers (conv1–conv3) followed by two fully connected layers (FC4/FC5), with four output heads for alternative allele, zygosity, variant type, and indel length, using SELU activations and He initialization (~1.63M parameters total). Load GIAB HG001 v3.3.2-derived training tensors and partition samples 90% training / 10% validation. Train until validation loss converges, raising the learning rate if training stalls early at a local optimum — the paper notes training 'will stuck early at a local optimal and cannot be further optimized without a higher learning rate'. Save the model checkpoint plus a training report recording the parameter count, output-head shapes, and final validation loss. A PyTorch port (Clairvoyante-PyTroch) reproduces the same API with three conv layers, two hidden fully connected layers, four output layers, and He initialization, using NCHW tensor format.

## Related tools

- **Tensorflow** (Machine-learning framework on which the five-layer Clairvoyante network is built and trained) — https://github.com/aquaskyline/Clairvoyante
- **Clairvoyante** (The multi-task five-layer CNN variant caller itself; open-source with modules to train, utilize and visualize the model) — https://github.com/aquaskyline/Clairvoyante
- **HKU-BAL/Clairvoyante-PyTroch** (PyTorch reimplementation with the exact same APIs as the TensorFlow clairvoyante_v3.py model, using NCHW tensor permutation) — https://github.com/HKU-BAL/Clairvoyante-PyTroch
- **Blosc/python-blosc** (Python dependency of the Clairvoyante toolset used for tensor compression in the data-preparation/calling pipeline) — https://github.com/Blosc/python-blosc

## Examples

```
pip install tensorflow==1.9.0 blosc intervaltree==2.1.0 numpy && python -c 'from clairvoyante.clairvoyante_v3 import Net; m = Net()'
```

## Evaluation signals

- Parameter count matches ~1.63M for the assembled network
- The four output-head shapes match the four prediction heads (alternative allele, zygosity, variant type, indel length)
- Training on GIAB HG001 v3.3.2-derived tensors converges to a validation loss near the reported value
- Per-input-tensor visualization yields separate activation maps for input tensor, conv1, conv2, conv3, fc4, fc5, and output predictions
- Network reproduces reported accuracy levels (e.g., ~99.7% accuracy on known variants for Illumina NA12878) when validated on the 10% validation partition

## Limitations

- Model training can occasionally get stuck at a local optimum early in training and requires a higher learning rate to escape
- Network version 3 can only output one of two possible alternative alleles at a position; multi-allelic variants are excluded
- Base quality of sequencing reads is not considered in the current implementation
- The exact alternative allele is only provided for indel variants ≤4 bp due to scarcity of long indel training samples
- Indel detection, especially for ONT data, has substantial room for improvement (Indel F1-score remains around 50%)
- Cannot be applied to completely novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset

## Evidence

- [abstract] Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length"
- [intro] Tensorflow framework statement: "the code is built on top of the Tensorflow machine-learning framework"
- [discussion] 90/10 split of training samples: "We randomly partitioned all samples into 90% for training and 10% for validation"
- [discussion] Local-optimum training failure mode: "the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate"
- [readme] PyTorch architecture parity and He initialization: "The code initialises Clairvoyante with 3 convolutional layers, 2 hidden fully connected layers and 4 output layers. It 
specifies the parameters for these layers and it initialises the network's"
- [readme] Training requires GPU version of TensorFlow: "To train a new model, a high-end GPU and the GPU version of Tensorflow is needed."
