---
name: variant-caller-threshold-configuration
description: Use when calling germline SNVs and small indels genome-wide with Clairvoyante
  on HG001/NA12878 BAMs and you need to set the alternative allele frequency cutoff
  and training sampling configuration to match published metrics.
when_to_use_negative:
- Data are multi-allelic or require detection of two or more alternative alleles at
  a position — Clairvoyante's current design ignores such variants and network version
  3 outputs only one alternative allele per position.
- Calling somatic variants or trio-sample-based variants — these were not yet supported
  in this Clairvoyante version.
- Input alignments are long indel-dominated and you need exact alternative alleles
  >4 bp — Clairvoyante provides exact alleles only for indels ≤4 bp, and ONT indel
  F1 remains ~50%.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: The multi-task five-layer CNN variant caller being configured; called genome-wide
    per BAM with the 0.2 allele frequency cutoff
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework Clairvoyante is built on; CPU version suffices
    for calling, GPU version required for training
- name: RTG vcfeval
  role: Haplotype-aware evaluation of called VCFs against the GIAB v3.3.2 HG001 truth
    set to verify threshold configuration
- name: NGMLR
  role: Realignment of PacBio/ONT reads to GRCh37 decoy v5 prior to calling (v0.2.3)
- name: Clairvoyante-PyTroch
  role: PyTorch reimplementation with identical APIs to clairvoyante_v3.py for users
    preferring PyTorch
  repo: HKU-BAL/Clairvoyante-PyTroch
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:ce55bcae5c566d1cef9de883ddc22be97cbc35f912ca370b22d64d79e115e835
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Clairvoyante-PyTroch
  - Illumina
  - NGMLR
  - Oxford Nanopore
  - PacBio
  - RTG vcfeval
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# variant-caller-threshold-configuration

## Summary

Configure the Clairvoyante variant caller's calling and training parameters — notably the alternative allele frequency cutoff (0.2 for whole-genome HG001 evaluation) and truth-to-non-variant sample pairing — to reproduce its reported F1-scores across Illumina, PacBio, and Oxford Nanopore data.

## When to use

Use when calling germline SNVs and small indels genome-wide with Clairvoyante on HG001/NA12878 BAMs and you need to set the alternative allele frequency cutoff and training sampling configuration to match published metrics.

## When NOT to use

- Data are multi-allelic or require detection of two or more alternative alleles at a position — Clairvoyante's current design ignores such variants and network version 3 outputs only one alternative allele per position.
- Calling somatic variants or trio-sample-based variants — these were not yet supported in this Clairvoyante version.
- Input alignments are long indel-dominated and you need exact alternative alleles >4 bp — Clairvoyante provides exact alleles only for indels ≤4 bp, and ONT indel F1 remains ~50%.

## Inputs

- HG001/NA12878 BAM alignment files (Illumina downsampled to 50x, PacBio realigned with NGMLR v0.2.3 to GRCh37 decoy v5, ONT rel3 MinION 1D R9.4)
- Reference genome FASTA (GRCh37 decoy version 5)
- Genome-in-a-Bottle v3.3.2 HG001 truth VCF (evaluation only)
- Pre-trained Clairvoyante models (trainedModels.tbz)
- Truth variant training sets (e.g., GIAB HG002 Ashkenazim sample)

## Outputs

- Called variant VCFs per sequencing technology
- vcfeval precision/recall/F1 metrics (e.g., vcfeval_f1_scores.csv) per technology
- Trained Clairvoyante model checkpoint files

## How to apply

Run Clairvoyante (a TensorFlow five-layer CNN) genome-wide on each BAM with the alternative allele frequency cutoff set to 0.2 for whole-genome HG001 evaluation. During model training, pair each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites, excluding multi-allelic SNPs; do NOT pre-filter alignments by mapping quality, as the authors explicitly advise against it. Split training data 90%/10% for training/validation with cross-sample testing instead of a held-out test set. Use callVarBam.py (4 threads default) for BAM-based calling; PyPy is required for acceptable speed on dataPrepScripts and callVarBam.py since Tensorflow is not PyPy-compatible. Validate by comparing output VCFs against GIAB v3.3.2 HG001 truth with RTG vcfeval, expecting whole-genome F1 of 98.65% (Illumina 50x downsampled), 92.57% (PacBio NGMLR v0.2.3 on GRCh37 decoy v5), and 87.26% (ONT rel3 1D R9.4).

## Related tools

- **Clairvoyante** (The multi-task five-layer CNN variant caller being configured; called genome-wide per BAM with the 0.2 allele frequency cutoff) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework Clairvoyante is built on; CPU version suffices for calling, GPU version required for training)
- **RTG vcfeval** (Haplotype-aware evaluation of called VCFs against the GIAB v3.3.2 HG001 truth set to verify threshold configuration)
- **NGMLR** (Realignment of PacBio/ONT reads to GRCh37 decoy v5 prior to calling (v0.2.3))
- **Clairvoyante-PyTroch** (PyTorch reimplementation with identical APIs to clairvoyante_v3.py for users preferring PyTorch) — HKU-BAL/Clairvoyante-PyTroch

## Examples

```
pip install tensorflow==1.9.0 && pypy callVarBam.py --input HG001.bam --ref GRCh37.fa --calModel trainedModels --outPrefix hg001 --threads 4
```

## Evaluation signals

- Whole-genome F1-scores match 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore rel5) at the 0.2 allele frequency cutoff against GIAB v3.3.2 HG001 via RTG vcfeval.
- vcfeval output yields TP/FP/FN counts from which Precision, Recall, and F1 are computed per technology and written to a single table (vcfeval_f1_scores.csv).
- Slim/reduced-parameter Clairvoyante models should produce noticeably lower F1, confirming the full model configuration is in effect.
- Whole-genome variant calling completes in under 2 hours on a 28 CPU-core machine, indicating the calling pipeline (e.g., PyPy-accelerated callVarBam) is correctly set up.
- No alignments are discarded by mapping quality filtering — the caller receives unfiltered BAM input per the authors' recommendation.

## Limitations

- Multi-allelic SNPs are excluded from training, and variants with two or more alternative alleles are ignored by the current design.
- Base quality of sequencing reads is not considered in the current implementation.
- Clairvoyante cannot be applied to novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset.
- Indel detection, especially on ONT data, remains weak (indel F1-score around 50%); exact alternative alleles are only provided for indels ≤4 bp.
- Model training can get stuck at a local optimum early and requires a higher learning rate to escape; ONT data exist only for HG001, limiting cross-sample evaluation on ONT.

## Evidence

- [abstract] Clairvoyante achieves whole-genome F1-scores of 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore rel5): "98.65, 92.57, 87.26% F1-score for whole-genome analysis, using Illumina, PacBio, and Oxford Nanopore data, respectively"
- [discussion] Truth-variant/non-variant pairing strategy for training: "we paired each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites"
- [discussion] No mapping-quality filtering of alignments: "We do not suggest removing any alignment by their mapping quality"
- [discussion] Multi-allelic SNP exclusion from training: "In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training"
- [readme] Tensorflow dependency and model architecture: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type, zygosity, alternative allele and Indel length"
- [readme] PyPy requirement for calling scripts: "To guarantee a good user experience (good speed), pypy must be installed to run `callVarBam.py` (call variants from BAM)"
