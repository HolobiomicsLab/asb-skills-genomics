---
name: deep-neural-network-variant-calling
description: Use when calling SNVs and small indels from single-molecule sequencing
  alignments (especially high-error ONT/PacBio BAMs) where traditional variant callers
  fail due to indel-enriched sequencing error, or when benchmarking caller accuracy
  against a GIAB truth set.
when_to_use_negative:
- Calling somatic variants, trio-based variants, or variants with two or more alternative
  alleles — the current Clairvoyante design does not support these and network version
  3 outputs only one alternative allele per position
- Variant calling on completely novel sequencing technologies/chemistries lacking
  a high-quality GIAB-style training dataset
- Precise genotyping of long indels — exact alternative alleles are only provided
  for indels <= 4 bp and ONT indel F1 remains around 50%
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3474
tools:
- name: Clairvoyante
  role: Multi-task five-layer CNN variant caller that extracts four-channel pileup
    tensors and predicts variant type, zygosity, alternative allele and indel length
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which Clairvoyante is built; GPU build required
    for training, CPU sufficient for calling
- name: NGMLR
  role: Aligner used to realign PacBio and ONT reads to the GRCh37 decoy version 5
    reference before tensor extraction
- name: RTG vcfeval
  role: Genotype-aware evaluation of the Clairvoyante call set against the GIAB HG001
    v3.3.2 truth VCF to compute FPR, FNR, precision and F1
- name: Nanopolish
  role: Baseline ONT whole-genome variant caller used for comparison against Clairvoyante
    at 1KGp3 common sites
- name: Clairvoyante-PyTroch
  role: PyTorch reimplementation with the same APIs as the TensorFlow clairvoyante_v3
    model (3 conv layers, 2 hidden FC layers, 4 output layers)
  repo: https://github.com/HKU-BAL/Clairvoyante-PyTroch
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:e4f8406bbeda4f506eb3d089bc88883d06ad64d75940dfbc112c2637f0a8b53d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Clairvoyante-PyTroch
  - Illumina
  - NGMLR
  - Nanopolish
  - Oxford Nanopore
  - RTG vcfeval
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# deep-neural-network-variant-calling

## Summary

Applies Clairvoyante, a multi-task five-layer convolutional neural network built on TensorFlow, to call SNPs and small indels from single-molecule sequencing (Illumina, PacBio, Oxford Nanopore) read alignments by predicting variant type, zygosity, alternative allele and indel length from four-channel pileup tensors.

## When to use

Use when calling SNVs and small indels from single-molecule sequencing alignments (especially high-error ONT/PacBio BAMs) where traditional variant callers fail due to indel-enriched sequencing error, or when benchmarking caller accuracy against a GIAB truth set.

## When NOT to use

- Calling somatic variants, trio-based variants, or variants with two or more alternative alleles — the current Clairvoyante design does not support these and network version 3 outputs only one alternative allele per position
- Variant calling on completely novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset
- Precise genotyping of long indels — exact alternative alleles are only provided for indels <= 4 bp and ONT indel F1 remains around 50%

## Inputs

- BAM alignment of single-molecule sequencing reads (Illumina 50x downsampled, PacBio, or Oxford Nanopore rel3 R9.4 MinION reads aligned with NGMLR v0.2.3 to GRCh37 decoy v5)
- GIAB truth VCF for training/evaluation (version 3.3.2 for HG001)
- Trained Clairvoyante model checkpoint (e.g., 999-epoch Illumina or 1999-epoch ONT model from trainedModels.tbz)
- Optional site filter list such as 1KG phase 3 common variant sites

## Outputs

- Genotyped VCF of predicted SNPs and small indels with variant type, zygosity, alternative allele and indel length
- vcfeval summary metrics (FPR, FNR, precision, F1-score), e.g., written to chr1_benchmark_metrics.csv or vcfeval_summary.txt

## How to apply

Train or load a Clairvoyante TensorFlow checkpoint (Tensorflow >= 1.0.0; training requires the GPU build) on GIAB truth variants, pairing each truth variant with two non-variants sampled at non-ambiguous sites and splitting 90%/10% train/validation. Feed the read alignment (e.g., ONT reads realigned to GRCh37 decoy v5 with NGMLR v0.2.3, or Illumina downsampled to 50x) to callVarBam.py, which extracts pileup tensors and predicts variant type, zygosity, alternative allele and indel length. Restrict the emitted VCF to a target site list such as the 1KG phase 3 common variants, then evaluate genotype-aware with RTG vcfeval against GIAB v3.3.2 truth, computing FPR, FNR, precision and F1 (e.g., best ONT chr1 configuration: 1999-epoch model giving F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55%; best Illumina: 999-epoch model giving F1 99.67%). Do not remove alignments by mapping quality, and use PyPy for the data-prep scripts and callVarBam.py since TensorFlow itself is not PyPy-compatible.

## Related tools

- **Clairvoyante** (Multi-task five-layer CNN variant caller that extracts four-channel pileup tensors and predicts variant type, zygosity, alternative allele and indel length) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which Clairvoyante is built; GPU build required for training, CPU sufficient for calling)
- **NGMLR** (Aligner used to realign PacBio and ONT reads to the GRCh37 decoy version 5 reference before tensor extraction)
- **RTG vcfeval** (Genotype-aware evaluation of the Clairvoyante call set against the GIAB HG001 v3.3.2 truth VCF to compute FPR, FNR, precision and F1)
- **Nanopolish** (Baseline ONT whole-genome variant caller used for comparison against Clairvoyante at 1KGp3 common sites)
- **Clairvoyante-PyTroch** (PyTorch reimplementation with the same APIs as the TensorFlow clairvoyante_v3 model (3 conv layers, 2 hidden FC layers, 4 output layers)) — https://github.com/HKU-BAL/Clairvoyante-PyTroch

## Examples

```
git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git && cd Clairvoyante && curl http://www.bio8.cs.hku.hk/trainedModels.tbz | tar -jxf - && python callVarBam.py --bam hg001_ont_chr1.bam --ref GRCh37.fa --outPrefix chr1_calls --chkfn trainedModels/ont_1999
```

## Evaluation signals

- Reproduced vcfeval metrics should match reported values: Illumina HG001 F1 99.67%, FPR 0.25%, FNR 0.41%, precision 99.75%; ONT chr1 F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55% (Nanopolish baseline F1 86.93%)
- PacBio/ONT whole-genome F1 should fall near the reported 92.57% / 87.26% and 1KP common-variant F1 near 95.78% / 90.53%
- Training run should confirm the 90/10 train/validation split and the 2:1 non-variant:variant pairing without training on multi-allelic SNPs
- Whole-genome calling should complete in under 2 hours on a 28-core CPU machine
- Slim/reduced-parameter model should produce a markedly lower F1 than the full five-layer model, confirming the full architecture was loaded

## Limitations

- Multi-allelic SNPs are excluded from training and variants with two or more alternative alleles are ignored; network version 3 outputs only one alternative allele per position
- Base quality of sequencing reads is not considered in the current implementation
- Exact alternative alleles are provided only for indels <= 4 bp; ONT indel F1 remains around 50%
- Training can get stuck at a local optimum early and may require a higher learning rate to escape
- Only ONT data for HG001 were available, limiting cross-sample evaluation and combined-sample training on ONT; application to novel chemistries is blocked absent a GIAB-style training set
- blosc installation may fail without AVX2 CPU support (compile with DISABLE_BLOSC_AVX2); the pure Python interpreter is slow and PyPy is needed for callVarBam.py and data-prep scripts

## Evidence

- [abstract] Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length"
- [discussion] Reads realigned with NGMLR to GRCh37 decoy v5: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] Training pairs each truth variant with two non-variants: "we paired each truth variant with two non-variants randomly sampled from the genome at all possible nonvariant and nonambiguous sites"
- [discussion] No mapping-quality filtering suggested: "We do not suggest removing any alignment by their mapping quality"
- [intro] Tensorflow framework: "the code is built on top of the Tensorflow machine-learning framework"
- [readme] Install Tensorflow and dependencies via pip: "pip install tensorflow==1.9.0
pip install blosc
pip install intervaltree==2.1.0"
- [readme] PyPy needed for callVarBam.py: "To guarantee a good user experience (good speed), pypy must be installed to run `callVarBam.py` (call variants from BAM)"
