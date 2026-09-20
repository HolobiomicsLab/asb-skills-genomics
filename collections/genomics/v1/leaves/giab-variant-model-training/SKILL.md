---
name: giab-variant-model-training
description: Use when building or reproducing a deep-learning variant caller that
  needs a trained model on GIAB truth variants, e.g. to genotype SNPs/indels from
  Illumina, PacBio, or Oxford Nanopore alignments.
when_to_use_negative:
- The target sequencing technology/chemistry has no high-quality GIAB-style training
  dataset available — Clairvoyante cannot be applied to completely novel technologies
  lacking such data
- The task is somatic or trio-sample-based variant calling — not supported by the
  current Clairvoyante implementation
- Variants of interest have two or more alternative alleles (multi-allelic SNPs) —
  excluded from training and ignored by the current design
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Clairvoyante
  role: Five-layer multitask CNN being reconstructed and trained; provides tensor
    encoding, network definition, training, and visualization modules
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante model code is built (>=
    1.0.0; GPU version required for training)
- name: HKU-BAL/Clairvoyante-PyTroch
  role: PyTorch reimplementation with the same APIs (clairvoyante_v3_pytorch.py) using
    3 conv layers, 2 hidden FC layers, 4 output layers, and He initialization
- name: Blosc/python-blosc
  role: Dependency for compressed tensor I/O in the training pipeline
  repo: https://github.com/Blosc/python-blosc
- name: nanopore-wgs-consortium/NA12878
  role: Source of Oxford Nanopore WGS reads (rel3, R9.4 chemistry) used to generate
    training tensors for HG001
  repo: https://github.com/nanopore-wgs-consortium/NA12878
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:1bf0f66c3851a0918aec5be08642c9eba28df5271f3a49caedb43c6be2c00ee8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Blosc/python-blosc
  - Clairvoyante
  - HKU-BAL/Clairvoyante-PyTroch
  - Tensorflow
  - nanopore-wgs-consortium/NA12878
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# giab-variant-model-training

## Summary

Reconstructs and trains the Clairvoyante five-layer multitask CNN (three conv layers plus two fully connected layers, four prediction heads) on tensors derived from Genome-in-a-Bottle (GIAB) HG001 v3.3.2 truth data for variant calling with single-molecule sequencing reads.

## When to use

Use when building or reproducing a deep-learning variant caller that needs a trained model on GIAB truth variants, e.g. to genotype SNPs/indels from Illumina, PacBio, or Oxford Nanopore alignments.

## When NOT to use

- The target sequencing technology/chemistry has no high-quality GIAB-style training dataset available — Clairvoyante cannot be applied to completely novel technologies lacking such data
- The task is somatic or trio-sample-based variant calling — not supported by the current Clairvoyante implementation
- Variants of interest have two or more alternative alleles (multi-allelic SNPs) — excluded from training and ignored by the current design

## Inputs

- GIAB Genome-in-a-Bottle v3.3.2 truth VCF for HG001 (NA12878)
- BAM/aligned reads from Illumina, PacBio, or Oxford Nanopore (e.g. ONT rel3 for NA12878)
- Reference genome (GRCh37 decoy version 5 or GRCh38)
- Encoded 33x4x4 training tensors produced by CreateTensor.py

## Outputs

- Trained Clairvoyante TensorFlow model checkpoint (~1.63M parameters)
- Training report with parameter count, four output-head shapes, and final validation loss
- Validation-loss convergence curve (90/10 train/validation split)

## How to apply

Encode each candidate variant locus as a 33x4x4 tensor spanning a 33 bp window (candidate position plus 16-bp flanks) with four channels: reference base, insertion count, deletion count, and alternate-allele count from the read alignment. Assemble the Clairvoyante v3 network in TensorFlow as three convolutional layers followed by fully connected layers FC4/FC5 with SELU activations and He initialization, terminating in four output heads (alternative allele, zygosity, variant type, indel length), totalling ~1.63M parameters. Load the GIAB HG001 v3.3.2-derived training tensors and randomly partition samples 90% training / 10% validation, pairing each truth variant with two non-variants sampled from nonvariant, nonambiguous sites; multi-allelic SNPs are excluded from training. Train until validation loss converges, and if training stalls early at a local optimum, raise the learning rate to escape it. Save the model checkpoint plus a training report recording parameter count, output-head shapes, and final validation loss.

## Related tools

- **Clairvoyante** (Five-layer multitask CNN being reconstructed and trained; provides tensor encoding, network definition, training, and visualization modules) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which the Clairvoyante model code is built (>= 1.0.0; GPU version required for training))
- **HKU-BAL/Clairvoyante-PyTroch** (PyTorch reimplementation with the same APIs (clairvoyante_v3_pytorch.py) using 3 conv layers, 2 hidden FC layers, 4 output layers, and He initialization)
- **Blosc/python-blosc** (Dependency for compressed tensor I/O in the training pipeline) — https://github.com/Blosc/python-blosc
- **nanopore-wgs-consortium/NA12878** (Source of Oxford Nanopore WGS reads (rel3, R9.4 chemistry) used to generate training tensors for HG001) — https://github.com/nanopore-wgs-consortium/NA12878

## Examples

```
pip install tensorflow==1.9.0 blosc intervaltree==2.1.0 numpy; python train.py --batchSize 500 --epoch 1 --vcf trainingData.vcf --bam trainingData.bam --ref ref.fa --outModel trainedModel
```

## Evaluation signals

- Parameter count of the assembled network matches ~1.63M
- Four output-head shapes match the four prediction tasks: alternative allele, zygosity, variant type, indel length
- Validation loss converges to a value near the reported figure on the 10% validation split of GIAB-derived tensors
- If training stalls early at a local optimum, raising the learning rate resumes loss decrease
- Trained model reproduces near-reported accuracy on NA12878 known variants (99.73%/97.68%/95.36% for Illumina/PacBio/ONT)

## Limitations

- Model training can get stuck early at a local optimum and cannot be further optimized without a higher learning rate
- Indel detection, especially on ONT data, remains weak (Indel F1-score around 50%)
- The network can only output one of two possible alternative alleles at a position, and exact alternative alleles only for indels <= 4 bp
- Base quality of sequencing reads is not considered in the current implementation
- Training requires a high-end GPU; 90/10 split means no held-out test set (cross-sample testing used instead)

## Evidence

- [abstract] multitask five-layer CNN architecture: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length"
- [intro] Tensorflow as underlying framework: "the code is built on top of the Tensorflow machine-learning framework"
- [discussion] 90/10 train/validation split: "We randomly partitioned all samples into 90% for training and 10% for validation"
- [discussion] two non-variants paired per truth variant: "we paired each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites"
- [discussion] local optimum training failure mode: "the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate"
- [readme] network initialization and structure: [evidence span withheld — anchor status: not_found]
- [readme] GPU needed for training: "To train a new model, a high-end GPU and the GPU version of Tensorflow is needed."
- [discussion] truth dataset version: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
