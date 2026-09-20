---
name: genome-wide-variant-calling-sms-reads
description: Use when you need to call germline SNPs/small indels genome-wide from
  BAM alignments of Illumina (50x), PacBio NGMLR-aligned, or ONT rel3 R9.4 reads,
  and evaluate Precision/Recall/F1 against a GIAB truth set.
when_to_use_negative:
- Your primary goal is accurate indel calling on ONT data — the Indel F1-score remains
  around 50% on ONT reads
- The truth variants contain multi-allelic sites or two or more alternative alleles
  at a position — Clairvoyante excludes multi-allelic SNPs from training and outputs
  only one of two possible alternative alleles
- The sequencing technology/chemistry is completely novel with no high-quality GIAB-style
  training dataset available — Clairvoyante training depends on such datasets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Clairvoyante
  role: TensorFlow-based multi-task five-layer CNN that predicts variant type, zygosity,
    alternative allele and Indel length from SMS BAM alignments genome-wide
  repo: https://github.com/aquaskyline/Clairvoyante
- name: RTG vcfeval
  role: Haplotype-aware genotype matching of Clairvoyante VCFs against the GIAB v3.3.2
    HG001 truth set to obtain TP/FP/FN counts
- name: NGMLR
  role: Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) before calling
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante code is built; CPU suffices
    for calling, GPU TensorFlow required for training
- name: Clair
  role: Successor of Clairvoyante for germline small variant calling on single-molecule
    sequencing data
  repo: https://github.com/HKU-BAL/Clair
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:b2e779ecf5ecf3c9c1db92772ce3058647ef5b59eb665d62312012c6bc489579
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
  - Clairvoyante
  - Illumina
  - NGMLR
  - Oxford Nanopore
  - PacBio
  - RTG vcfeval
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# genome-wide-variant-calling-sms-reads

## Summary

Genome-wide SNV and small indel calling on single-molecule sequencing reads (Illumina, PacBio, Oxford Nanopore) using Clairvoyante, a multi-task five-layer CNN built on TensorFlow, followed by haplotype-aware truth-set benchmarking with RTG vcfeval.

## When to use

Use when you need to call germline SNPs/small indels genome-wide from BAM alignments of Illumina (50x), PacBio NGMLR-aligned, or ONT rel3 R9.4 reads, and evaluate Precision/Recall/F1 against a GIAB truth set.

## When NOT to use

- Your primary goal is accurate indel calling on ONT data — the Indel F1-score remains around 50% on ONT reads
- The truth variants contain multi-allelic sites or two or more alternative alleles at a position — Clairvoyante excludes multi-allelic SNPs from training and outputs only one of two possible alternative alleles
- The sequencing technology/chemistry is completely novel with no high-quality GIAB-style training dataset available — Clairvoyante training depends on such datasets

## Inputs

- BAM alignment of Illumina reads downsampled to 50x (HG001/NA12878)
- BAM alignment of PacBio reads realigned to GRCh37 decoy version 5 with NGMLR v0.2.3
- BAM/FASTQ of Oxford Nanopore rel3 MinION 1D R9.4 reads (nanopore-wgs-consortium/NA12878)
- Genome-in-a-Bottle (GIAB) version 3.3.2 truth VCF for HG001
- Pre-trained Clairvoyante models (trainedModels.tbz, TensorFlow >= 1.0.0)

## Outputs

- Per-technology VCF callsets of SNPs and small indels
- vcfeval true-positive, false-positive, and false-negative counts per technology
- Precision, Recall, and F1-score table per sequencing technology (e.g. vcfeval_f1_scores.csv)

## How to apply

Load the three HG001/NA12878 BAM sets: Illumina downsampled to 50x coverage, PacBio reads realigned to GRCh37 decoy version 5 with NGMLR v0.2.3, and Oxford Nanopore rel3 MinION 1D R9.4 reads (do NOT remove alignments by mapping quality). Run Clairvoyante genome-wide on each BAM (callVarBam.py, 4 threads by default via --threads) with the alternative allele frequency cutoff set to 0.2 to produce VCF callsets. Compare each VCF against the Genome-in-a-Bottle version 3.3.2 truth dataset for HG001 using RTG vcfeval for haplotype-aware genotype matching, extract true-positive/false-positive/false-negative counts, and compute Precision, Recall, and F1 per technology. Rationale: Clairvoyante's five-layer CNN disentangles subtle read-alignment details that traditional callers fail to handle at SMS error rates; a slim reduced-parameter model produces a much lower F1, so the full model must be used.

## Related tools

- **Clairvoyante** (TensorFlow-based multi-task five-layer CNN that predicts variant type, zygosity, alternative allele and Indel length from SMS BAM alignments genome-wide) — https://github.com/aquaskyline/Clairvoyante
- **RTG vcfeval** (Haplotype-aware genotype matching of Clairvoyante VCFs against the GIAB v3.3.2 HG001 truth set to obtain TP/FP/FN counts)
- **NGMLR** (Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) before calling)
- **Tensorflow** (Machine-learning framework on which the Clairvoyante code is built; CPU suffices for calling, GPU TensorFlow required for training)
- **Clair** (Successor of Clairvoyante for germline small variant calling on single-molecule sequencing data) — https://github.com/HKU-BAL/Clair

## Examples

```
git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git && cd Clairvoyante && curl http://www.bio8.cs.hku.hk/trainedModels.tbz | tar -jxf - && pip install tensorflow==1.9.0 blosc intervaltree==2.1.0 numpy
```

## Evaluation signals

- Whole-genome F1-scores against GIAB v3.3.2 HG001 must match 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore rel5/rel3)
- vcfeval output must yield TP/FP/FN counts from which Precision = TP/(TP+FP), Recall = TP/(TP+FN), F1 = 2PR/(P+R) reproduce those scores
- Runtime invariant: whole-genome variant calling completes in less than 2 hours on a standard 28 CPU-core machine
- A slim (reduced-parameter) model should produce a much lower F1 than the full model — confirms full-model feature extraction was used
- All metrics for the three technologies land in a single table (vcfeval_f1_scores.csv) with one row per technology

## Limitations

- Indel detection, especially for ONT data, has an Indel F1-score around 50% and needs substantial improvement
- Multi-allelic SNPs are excluded from training and the network can only output one of two possible alternative alleles at a position
- Base quality of sequencing reads is not considered in the current implementation
- Exact alternative allele is only provided for indel variants <=4 bp due to scarcity of long indel training samples
- Cannot be applied to completely novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset; ONT data were only available for HG001, limiting cross-sample evaluation
- Model training can get stuck early at a local optimum and requires a higher learning rate to escape

## Evidence

- [abstract] Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis, using Illumina, PacBio, and Oxford Nanopore data, respectively: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [methods] Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] We do not suggest removing any alignment by their mapping quality: "We do not suggest removing any alignment by their mapping quality"
- [readme] achieved 98.65%, 92.57%, 87.26% F1 score on the whole genome, in Illumina, PacBio, and Oxford Nanopore data, respectively: "achieved 98.65%, 92.57%, 87.26% F1 score on the whole genome, in Illumina, PacBio, and Oxford Nanopore data, respectively"
