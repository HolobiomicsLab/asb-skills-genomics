---
name: genomic-variant-tensor-encoding
description: Use when calling SNVs/small indels from Illumina, PacBio or Oxford Nanopore
  (SMS) BAMs with a deep learning caller, and you must convert pileups at candidate
  loci into fixed-size numeric tensors for a CNN.
when_to_use_negative:
- The candidate locus carries two or more alternative alleles — the current Clairvoyante
  design ignores multi-allelic variants and multi-allelic SNPs are excluded from training
- You need base-quality-aware calling — the current implementation does not consider
  sequencing read base quality
- The indel variant is longer than 4 bp — exact alternative allele is only provided
  for indel variants ≤4 bp due to scarce long-indel training samples
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Clairvoyante
  role: Multitask five-layer CNN (TensorFlow) that consumes the encoded tensors and
    outputs variant type, zygosity, alternative allele and indel length predictions
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante code and model training
    are built
- name: HKU-BAL/Clairvoyante-PyTroch
  role: 'PyTorch reimplementation of the same network: 3 convolutional layers, 2 hidden
    fully connected layers, 4 output layers, He initialization; requires NCHW tensor
    permutation'
  repo: https://github.com/HKU-BAL/Clairvoyante-PyTroch
- name: Clair
  role: Successor program to Clairvoyante for germline small variant calling with
    SMS data
  repo: https://github.com/HKU-BAL/Clair
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:1a38b41214974127cc0db5a22f6092e3a5d53973a09d10468c7404c4804c539d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# genomic-variant-tensor-encoding

## Summary

Encodes read alignments around candidate variant loci into fixed-size 33x4x4 tensors (33 bp window with 16-bp flanks; four channels: reference base, insertion count, deletion count, alternate-allele count) suitable for the Clairvoyante five-layer multitask CNN that predicts variant type, zygosity, alternative allele and indel length.

## When to use

Use when calling SNVs/small indels from Illumina, PacBio or Oxford Nanopore (SMS) BAMs with a deep learning caller, and you must convert pileups at candidate loci into fixed-size numeric tensors for a CNN.

## When NOT to use

- The candidate locus carries two or more alternative alleles — the current Clairvoyante design ignores multi-allelic variants and multi-allelic SNPs are excluded from training
- You need base-quality-aware calling — the current implementation does not consider sequencing read base quality
- The indel variant is longer than 4 bp — exact alternative allele is only provided for indel variants ≤4 bp due to scarce long-indel training samples

## Inputs

- BAM/aligned read data from Illumina, PacBio or Oxford Nanopore sequencing (e.g. NA12878 ONT rel3 reads)
- Truth variant set (e.g. Genome-in-a-Bottle v3.3.2 for HG001) for label generation
- Reference genome (e.g. GRCh37 decoy version 5)

## Outputs

- 33x4x4 encoded tensors per candidate locus (channels: reference base, insertion count, deletion count, alternate-allele count)
- Four predictions per locus: alternative allele, zygosity, variant type (SNP or indel), indel length
- Trained TensorFlow model checkpoint (~1.63M parameters) and training report with validation loss

## How to apply

For each candidate variant locus, build a 33x4x4 input tensor spanning a 33 bp window (candidate position plus 16 bp on each side), filling four channels with reference base, insertion count, deletion count and alternate-allele count from the read alignment. Feed these tensors into the Clairvoyante network: three convolutional layers (conv1–conv3) followed by two fully connected layers (fc4, fc5) with SELU activations and He initialization, totalling ~1.63M parameters, ending in four prediction heads (alternative allele, zygosity, variant type, indel length). Train on GIAB HG001 v3.3.2-derived tensors partitioned 90% training / 10% validation; if training stalls early at a local optimum, raise the learning rate, since the paper notes training 'cannot be further optimized without a higher learning rate'. The repo's visualization tooling should render separate activations for input tensors, conv1, conv2, conv3, fc4, fc5 and output predictions, confirming layer-level data flow.

## Related tools

- **Clairvoyante** (Multitask five-layer CNN (TensorFlow) that consumes the encoded tensors and outputs variant type, zygosity, alternative allele and indel length predictions) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which the Clairvoyante code and model training are built)
- **HKU-BAL/Clairvoyante-PyTroch** (PyTorch reimplementation of the same network: 3 convolutional layers, 2 hidden fully connected layers, 4 output layers, He initialization; requires NCHW tensor permutation) — https://github.com/HKU-BAL/Clairvoyante-PyTroch
- **Clair** (Successor program to Clairvoyante for germline small variant calling with SMS data) — https://github.com/HKU-BAL/Clair

## Examples

```
pip install tensorflow==1.9.0 blosc intervaltree==2.1.0 numpy; git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git && cd Clairvoyante && curl http://www.bio8.cs.hku.hk/trainedModels.tbz | tar -jxf -
```

## Evaluation signals

- Parameter count of the assembled network matches ~1.63M
- Network exposes four output heads with shapes matching alternative allele, zygosity, variant type and indel length
- Training on GIAB HG001 v3.3.2-derived tensors converges to a validation loss near the reported value on the 10% validation split
- Visualization tooling renders distinct activations for input tensors, conv1, conv2, conv3, fc4, fc5 and output predictions
- On NA12878, trained model reaches ~99.67%, 95.78%, 90.53% F1-score on 1KP common variants for Illumina, PacBio and ONT data respectively

## Limitations

- Model training can get stuck early at a local optimum and cannot be further optimized without a higher learning rate
- Cannot be applied to completely novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset
- Indel detection for ONT data needs substantial improvement (Indel F1-score remains around 50%)
- Network version 3 can only output one of two possible alternative alleles at a position; multi-allelic SNPs are excluded from training and base quality is not considered

## Evidence

- [abstract] we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length"
- [readme] The code initialises Clairvoyante with 3 convolutional layers, 2 hidden fully connected layers and 4 output layers.: [evidence span withheld — anchor status: external_doc]
- [discussion] We randomly partitioned all samples into 90% for training and 10% for validation: "We randomly partitioned all samples into 90% for training and 10% for validation"
- [discussion] the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate: "the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate"
- [abstract] Clairvoyante is available open-source ( https://github.com/aquaskyline/Clairvoyante ), with modules to train, utilize and visualize the model.: "Clairvoyante is available open-source ( https://github.com/aquaskyline/Clairvoyante ), with modules to train, utilize and visualize the model."
