---
name: transfer-learning-model-fine-tuning
description: Use when a pre-trained variant-calling network (DeepVariant or Clairvoyante)
  must be adapted to a new sequencing platform, sample, or chromosome — e.g., an Illumina-trained
  or chr21-trained ONT model re-applied to new alignments and validated against a
  GIAB truth set.
when_to_use_negative:
- The sequencing technology or chemistry is completely novel and no high-quality GIAB-style
  training dataset on standard samples exists — the paper states this hinders Clairvoyante
  from being applied at all.
- 'Only inference is needed: a downloaded trained model already covers the platform/sample,
  so running callVarBam.py-style calling requires no fine-tuning.'
- Target events are multi-allelic or long indels — Clairvoyante's current design ignores
  variants with two or more alternative alleles and provides exact alternative alleles
  only for indels ≤4 bp.
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Clairvoyante
  role: Multi-task five-layer convolutional network whose pre-trained models are downloaded
    and fine-tuned across samples and platforms; training scripts and visualization
    toolset live in the repo
  repo: https://github.com/aquaskyline/Clairvoyante
- name: DeepVariant
  role: Inception-based caller benchmarked with transfer learning from the pre-trained
    Illumina model and the chr21-trained ONT model following the coauthor's guide
- name: Tensorflow
  role: Machine-learning framework Clairvoyante is built on; the GPU build is required
    for fine-tuning while CPU suffices for calling
- name: Clairvoyante-PyTroch
  role: PyTorch reimplementation exposing the exact same APIs as clairvoyante_v3.py,
    usable as an alternative backend for fine-tuning with one or more GPUs
  repo: https://github.com/HKU-BAL/Clairvoyante-PyTroch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:8ed82b1d146b9d73904dd13e39087b1b7dccfbd2308783435fe4c445241d0273
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Clairvoyante-PyTroch
  - DeepVariant
  - Illumina
  - Oxford Nanopore
  - Tensorflow
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# transfer-learning-model-fine-tuning

## Summary

Fine-tunes a pre-trained deep neural network variant caller (Clairvoyante's multi-task five-layer CNN or DeepVariant's inception-based model) by transferring weights from a model trained on one platform, sample, or chromosome and re-training on the target data, with accuracy benchmarked against the GIAB v3.3.2 HG001 truth set.

## When to use

Use when a pre-trained variant-calling network (DeepVariant or Clairvoyante) must be adapted to a new sequencing platform, sample, or chromosome — e.g., an Illumina-trained or chr21-trained ONT model re-applied to new alignments and validated against a GIAB truth set.

## When NOT to use

- The sequencing technology or chemistry is completely novel and no high-quality GIAB-style training dataset on standard samples exists — the paper states this hinders Clairvoyante from being applied at all.
- Only inference is needed: a downloaded trained model already covers the platform/sample, so running callVarBam.py-style calling requires no fine-tuning.
- Target events are multi-allelic or long indels — Clairvoyante's current design ignores variants with two or more alternative alleles and provides exact alternative alleles only for indels ≤4 bp.

## Inputs

- Pre-trained model checkpoint (Clairvoyante trainedModels.tbz, or DeepVariant pre-trained Illumina model / chr21-trained ONT model)
- BAM alignments for the target data type (Illumina downsampled to 50×; Oxford Nanopore rel5 alignments)
- GIAB truth-set VCF, version 3.3.2 for HG001
- Reference genome FASTA (GRCh37 decoy version 5, aligned with NGMLR 0.2.3 for SMS reads; GRCh38 also used)
- High-end GPU with the GPU build of Tensorflow for training

## Outputs

- Fine-tuned model checkpoint for the target platform, sample, or chromosome
- Variant call VCFs on the target alignments (e.g., ONT chr1/chr22 calls)
- Benchmark summary CSV of precision/recall/F1 per dataset and chromosome (deepvariant_clairvoyante_benchmark.csv)

## How to apply

Start from a pre-trained checkpoint — the Clairvoyante trainedModels.tbz bundle (fetched from bio8.cs.hku.hk) or DeepVariant's pre-trained Illumina model per the coauthor's transfer-learning guide — and fine-tune on tensors built from the target platform's BAM (e.g., ONT rel5 alignments, or Illumina downsampled to 50× to match SMS coverage). Prepare training pairs per the paper: each GIAB truth variant is paired with two non-variants randomly sampled at all non-variant, non-ambiguous sites; exclude multi-allelic SNPs and keep all alignments regardless of mapping quality; partition samples 90% training / 10% validation. Fine-tuning requires a high-end GPU with tensorflow-gpu (CPU-only installs suffice only for calling); the paper warns that training can get stuck early at a local optimum and cannot be further optimized without raising the learning rate. Choose the transfer scope deliberately: cross-sample transfer (train on HG002, call NA12878) demonstrates sample-agnostic behavior, while the DeepVariant benchmark transferred the chr21-trained ONT model to chr1/chr22. Validate by computing precision, recall, and F1 against GIAB v3.3.2 HG001: expect Illumina precision 0.9995 / recall 0.9991 and ONT F1 of 83.05% (chr1) and 77.89% (chr22) within rounding tolerance, written to deepvariant_clairvoyante_benchmark.csv.

## Related tools

- **Clairvoyante** (Multi-task five-layer convolutional network whose pre-trained models are downloaded and fine-tuned across samples and platforms; training scripts and visualization toolset live in the repo) — https://github.com/aquaskyline/Clairvoyante
- **DeepVariant** (Inception-based caller benchmarked with transfer learning from the pre-trained Illumina model and the chr21-trained ONT model following the coauthor's guide)
- **Tensorflow** (Machine-learning framework Clairvoyante is built on; the GPU build is required for fine-tuning while CPU suffices for calling)
- **Clairvoyante-PyTroch** (PyTorch reimplementation exposing the exact same APIs as clairvoyante_v3.py, usable as an alternative backend for fine-tuning with one or more GPUs) — https://github.com/HKU-BAL/Clairvoyante-PyTroch

## Examples

```
git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git && cd Clairvoyante && curl http://www.bio8.cs.hku.hk/trainedModels.tbz | tar -jxf -
```

## Evaluation signals

- Fine-tuned Illumina model reproduces precision 0.9995 and recall 0.9991 against GIAB v3.3.2 HG001 within rounding tolerance
- Model transferred from the chr21-trained checkpoint reaches F1 83.05% on ONT chr1 and 77.89% on ONT chr22
- Training setup honors the paper's protocol: 90/10 train/validation split, two non-variants paired per truth variant, multi-allelic SNPs excluded, no mapping-quality filtering of alignments
- Cross-sample sanity check: a model trained on one human sample (HG002) calls NA12878 with whole-genome F1 near 98.65/92.57/87.26% on Illumina/PacBio/ONT, confirming sample-agnostic transfer
- No early training stall — if the model gets stuck at a local optimum, increasing the learning rate must let optimization resume

## Limitations

- Transfer depends on a high-quality sequencing dataset on standard samples such as GIAB; completely novel sequencing technologies and chemistries cannot be covered.
- Training can get stuck early at a local optimum and cannot be further optimized without a higher learning rate.
- Only ONT data for HG001 were available, limiting cross-sample variant calling evaluation and combined-sample training on ONT data.
- Clairvoyante v3 ignores multi-allelic variants, gives exact alleles only for indels ≤4 bp, does not consider base quality, and its ONT indel F1 remains around 50%.
- Fine-tuning requires a high-end GPU with the GPU version of Tensorflow; CPU-only installations support calling only.

## Evidence

- [abstract] Training on a second human sample shows Clairvoyante is sample agnostic: "Training on a second human sample shows Clairvoyante is sample agnostic"
- [discussion] we paired each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites: "we paired each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites"
- [discussion] We randomly partitioned all samples into 90% for training and 10% for validation: "We randomly partitioned all samples into 90% for training and 10% for validation"
- [discussion] the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate: "the model training will stuck early at a local optimal and cannot be further optimized without a higher learning rate"
- [discussion] This hinders Clairvoyante from being applied to completely novel sequencing technologies and chemistries: "This hinders Clairvoyante from being applied to completely novel sequencing technologies and chemistries"
- [readme] To train a new model, a high-end GPU and the GPU version of Tensorflow is needed: "To train a new model, a high-end GPU and the GPU version of Tensorflow is needed"
