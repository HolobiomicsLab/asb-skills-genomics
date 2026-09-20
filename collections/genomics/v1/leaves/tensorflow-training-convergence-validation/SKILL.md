---
name: tensorflow-training-convergence-validation
description: Use when rebuilding a deep variant caller's neural network in TensorFlow
  — specifically when input tensors (e.g. 33x4x4 per locus) must be trained against
  labeled truth variants and checked for convergence and architecture fidelity.
when_to_use_negative:
- If your goal is variant calling with an already-trained model (CPU suffices); training
  a new model requires a high-end GPU and GPU TensorFlow
- If the input is not tensorized candidate loci with truth labels — e.g. you only
  have raw reads and no GIAB-style high-quality training dataset for the sequencing
  chemistry
- If your truth set contains multi-allelic SNPs; these are excluded from training
  in the current Clairvoyante implementation
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0625
tools:
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante training code is built;
    GPU version required for training new models
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Clairvoyante
  role: Multitask five-layer CNN variant caller whose v3 architecture is reconstructed
    and trained; includes modules to train, utilize and visualize the model
  repo: https://github.com/aquaskyline/Clairvoyante
- name: HKU-BAL/Clairvoyante-PyTroch
  role: PyTorch reimplementation with the same APIs as the TensorFlow v3 model (3
    conv layers, 2 hidden FC layers, 4 output layers, He initialization)
  repo: https://github.com/HKU-BAL/Clairvoyante-PyTroch
- name: nanopore-wgs-consortium/NA12878
  role: Source of Oxford Nanopore WGS reads (rel3) for generating training/evaluation
    tensors
  repo: https://github.com/nanopore-wgs-consortium/NA12878
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:7cb1bbab6487f77b43bf6530fb1d0aca31a779f5b968bd286893a684ebb51ffe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - HKU-BAL/Clairvoyante-PyTroch
  - Tensorflow
  - nanopore-wgs-consortium/NA12878
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tensorflow-training-convergence-validation

## Summary

Reconstruct and train the Clairvoyante five-layer multitask CNN (three conv layers plus FC4/FC5) in TensorFlow until validation loss converges on GIAB HG001 v3.3.2-derived tensors, verifying the ~1.63M parameter count and the four prediction heads (alternative allele, zygosity, variant type, indel length).

## When to use

Use when rebuilding a deep variant caller's neural network in TensorFlow — specifically when input tensors (e.g. 33x4x4 per locus) must be trained against labeled truth variants and checked for convergence and architecture fidelity.

## When NOT to use

- If your goal is variant calling with an already-trained model (CPU suffices); training a new model requires a high-end GPU and GPU TensorFlow
- If the input is not tensorized candidate loci with truth labels — e.g. you only have raw reads and no GIAB-style high-quality training dataset for the sequencing chemistry
- If your truth set contains multi-allelic SNPs; these are excluded from training in the current Clairvoyante implementation

## Inputs

- Training tensors: 33x4x4 per candidate locus (reference base, insertion count, deletion count, alternate-allele count channels) from read alignments (BAM)
- Truth variant labels derived from Genome-in-a-Bottle (GIAB) v3.3.2 for HG001
- Reference genome (GRCh37 decoy version 5 used for alignments)

## Outputs

- Trained TensorFlow model checkpoint (Clairvoyante v3, ~1.63M parameters)
- Training report with parameter count, four output-head shapes, and final validation loss
- Trained model files (e.g. trainedModels.tbz distribution) usable for visualization and variant calling

## How to apply

Encode each candidate variant locus as a 33x4x4 tensor spanning a 33 bp window (candidate position plus 16-bp flanks on each side) with four channels: reference base, insertion count, deletion count and alternate-allele count from the read alignment. Assemble the Clairvoyante v3 network in TensorFlow: three convolutional layers followed by two fully connected layers (FC4/FC5) with SELU activations, He weight initialization, and four output heads, totaling ~1.63M parameters. Load the GIAB HG001 v3.3.2-derived training tensors and randomly partition samples 90% training / 10% validation. Train until the validation loss converges near the reported value; if training stalls early at a local optimum, raise the learning rate to escape. Save the trained checkpoint plus a training report recording parameter count, output-head shapes, and final validation loss. A slim version with reduced parameters produces much lower F1, so preserve the full architecture rather than shrinking layers.

## Related tools

- **Tensorflow** (Machine-learning framework on which the Clairvoyante training code is built; GPU version required for training new models) — https://github.com/aquaskyline/Clairvoyante
- **Clairvoyante** (Multitask five-layer CNN variant caller whose v3 architecture is reconstructed and trained; includes modules to train, utilize and visualize the model) — https://github.com/aquaskyline/Clairvoyante
- **HKU-BAL/Clairvoyante-PyTroch** (PyTorch reimplementation with the same APIs as the TensorFlow v3 model (3 conv layers, 2 hidden FC layers, 4 output layers, He initialization)) — https://github.com/HKU-BAL/Clairvoyante-PyTroch
- **nanopore-wgs-consortium/NA12878** (Source of Oxford Nanopore WGS reads (rel3) for generating training/evaluation tensors) — https://github.com/nanopore-wgs-consortium/NA12878

## Examples

```
pip install tensorflow-gpu; pip install blosc; pip install intervaltree==2.1.0  # GPU TensorFlow required to train a new Clairvoyante model
```

## Evaluation signals

- Total trainable parameter count matches ~1.63M for the full (not slim) model
- The network exposes exactly four output heads matching the four predictions: alternative allele, zygosity, variant type (SNP or indel), indel length
- Random 90% training / 10% validation split is reproduced with no held-out test set inside the training sample
- Final validation loss converges near the reported value; verify the run did not stall early at a local optimum (which requires raising the learning rate)
- Layer inventory matches v3: conv1–conv3 plus FC4/FC5, with SELU activations and He initialization

## Limitations

- Model training can get stuck early at a local optimum and cannot be further optimized without a higher learning rate
- Training requires a high-end GPU with the GPU version of TensorFlow; the pure Python interpreter is slow and PyPy only accelerates Tensorflow-independent modules (dataPrepScripts, callVarBam.py)
- Clairvoyante v3 can output only one of the two possible alternative alleles at a position, and exact alternative alleles are limited to indel variants ≤4 bp due to scarce long-indel training samples
- Base quality of sequencing reads is not considered in the current implementation

## Evidence

- [discussion] Training data split 90% training / 10% validation with no held-out test set: "We randomly partitioned all samples into 90% for training and 10% for validation"
- [discussion] Training can stall at a local optimum requiring a higher learning rate: "the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate"
- [readme] Training requires GPU TensorFlow: "To train a new model, a high-end GPU and the GPU version of Tensorflow is needed."
- [readme] PyTorch port confirms the v3 architecture and initialization: [evidence span withheld — anchor status: external_doc]
- [intro] Model is built on TensorFlow with modules to train and visualize: "the code is built on top of the Tensorflow machine-learning framework"
