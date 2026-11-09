---
name: common-variant-site-restriction
description: Use when benchmarking germline variant calls (SNPs/indels) in HG001/NA12878
  against the paper's 1KP common-variant metrics (F1 99.67% Illumina, 95.78% PacBio,
  90.53% ONT) rather than whole-genome figures (98.65/92.57/87.26%).
when_to_use_negative:
- Evaluating whole-genome performance — use full-genome benchmarking instead, which
  the paper reports separately (F1 98.65/92.57/87.26% for Illumina/PacBio/ONT)
- Multi-allelic or multi-allelic-SNP sites — the current Clairvoyante implementation
  excludes multi-allelic SNPs from training and ignores variants with two or more
  alternative alleles
- Indel events longer than 4 bp — Clairvoyante only provides the exact alternative
  allele for indels ≤4 bp, so calls at such sites will lack exact alleles
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Tensorflow-based five-layer convolutional neural network that extracts four-channel
    pileup tensors and predicts variant type, zygosity, alternative allele and indel
    length at candidate sites
  repo: https://github.com/aquaskyline/Clairvoyante
- name: RTG vcfeval
  role: Genotype-aware benchmarking of restricted VCF calls against the GIAB v3.3.2
    HG001 truth dataset to classify true/false positives and negatives
- name: NGMLR
  role: Re-aligns PacBio (and ONT) reads to GRCh37 decoy version 5 before calling,
    version 0.2.3
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:e37f3dd5b3bb80ea25b9bd8cda999fd0de5419b25a10c13111bbf61c101e0131
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - NGMLR
  - PacBio
  - RTG vcfeval
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# common-variant-site-restriction

## Summary

Restricts variant caller output (e.g. Clairvoyante VCF calls on HG001) to the 1000 Genomes Project phase 3 (1KGp3) common variant site list before benchmarking, enabling comparison against published 1KP common-variant accuracy figures (99.67% Illumina, 95.78% PacBio F1-score).

## When to use

Use when benchmarking germline variant calls (SNPs/indels) in HG001/NA12878 against the paper's 1KP common-variant metrics (F1 99.67% Illumina, 95.78% PacBio, 90.53% ONT) rather than whole-genome figures (98.65/92.57/87.26%).

## When NOT to use

- Evaluating whole-genome performance — use full-genome benchmarking instead, which the paper reports separately (F1 98.65/92.57/87.26% for Illumina/PacBio/ONT)
- Multi-allelic or multi-allelic-SNP sites — the current Clairvoyante implementation excludes multi-allelic SNPs from training and ignores variants with two or more alternative alleles
- Indel events longer than 4 bp — Clairvoyante only provides the exact alternative allele for indels ≤4 bp, so calls at such sites will lack exact alleles

## Inputs

- Clairvoyante-trained model checkpoint (e.g. HG001 Illumina 999 epochs or HG001 PacBio 1499 epochs, Tensorflow-based)
- Sequencing alignment BAM/CRAM (50x downsampled GIAB Illumina HG001, or 44x PacBio HG001 aligned to GRCh37 decoy v5 with NGMLR v0.2.3)
- 1KG project phase 3 common variant site list (BED or VCF-format interval file)
- Genome-in-a-Bottle v3.3.2 HG001 truth VCF for RTG vcfeval

## Outputs

- VCF of genotyped variant calls restricted to 1KGp3 common variant sites
- vcfeval summary with true positives, false positives, false negatives (e.g. vcfeval_summary.txt or vcfeval_performance_metrics.tsv)
- Computed FPR, FNR, precision and F1-score at common variant sites

## How to apply

Run the caller (e.g. Clairvoyante callVarBam.py on a BAM such as the 50x downsampled GIAB Illumina HG001 alignment or the 44x PacBio HG001 alignment realigned to GRCh37 decoy v5 with NGMLR v0.2.3) to produce a VCF, then restrict emitted calls to the 1KG project phase 3 common variant site list and write the genotyped calls to a VCF. Benchmark with RTG vcfeval using the Genome-in-a-Bottle version 3.3.2 truth dataset for HG001 as baseline, stratified to the same 1KGp3 common-variant regions, and compute FPR, FNR, precision and F1-score genotype-aware. Expect the best Illumina configuration (HG001-trained model at 999 epochs) to reproduce FPR 0.25%, FNR 0.41%, precision 99.75%, F1 99.67%; the best PacBio configuration (1499-epoch HG001 model) to reproduce FPR 2.17%, FNR 6.06%, precision 97.70%, F1 95.78%. Discard indel calls longer than 4 bp, since the model cannot provide an exact alternative allele for them.

## Related tools

- **Clairvoyante** (Tensorflow-based five-layer convolutional neural network that extracts four-channel pileup tensors and predicts variant type, zygosity, alternative allele and indel length at candidate sites) — https://github.com/aquaskyline/Clairvoyante
- **RTG vcfeval** (Genotype-aware benchmarking of restricted VCF calls against the GIAB v3.3.2 HG001 truth dataset to classify true/false positives and negatives)
- **NGMLR** (Re-aligns PacBio (and ONT) reads to GRCh37 decoy version 5 before calling, version 0.2.3)

## Examples

```
python callVarBam.py --input ${BAM}.bam --out ${SAMPLE}_1kgp3.vcf && rtg vcfeval --baseline giab_3.3.2_HG001.vcf.gz --calls ${SAMPLE}_1kgp3.vcf --output vcfeval_summary
```

## Evaluation signals

- Illumina common-variant metrics match Table 2: FPR 0.25%, FNR 0.41%, precision 99.75%, F1-score 99.67% (HG001-trained model, 999 epochs)
- PacBio common-variant metrics match Table 3: FPR 2.17%, FNR 6.06%, precision 97.70%, F1-score 95.78% (HG001-trained model, 1499 epochs)
- Every variant in the output VCF overlaps a 1KGp3 common variant site (interval intersection check)
- vcfeval summary reports genotype-aware TP/FP/FN counts, and FPR + precision sum to ~100% (0.25% FPR ↔ 99.75% precision)
- No indel call in the output has an exact alternative allele for an indel >4 bp

## Limitations

- Results are only comparable to the paper's figures if the same truth set (GIAB v3.3.2 for HG001), the same region restriction (1KGp3 common variants), and the same reference (GRCh37 decoy v5) are used
- Multi-allelic SNPs are excluded from training and variants with two or more alternative alleles are ignored by the current design
- Indel detection remains weak, especially for ONT data where the Indel F1-score remains around 50%
- Base quality of sequencing reads is not considered by the caller

## Evidence

- [other] Restrict the emitted calls to the 1KG project phase 3 common variant site list and write the genotyped calls to a VCF.: [evidence span withheld — anchor status: not_found]
- [abstract] Clairvoyante achieves 99.67, 95.78, and 90.53% F1-scores on 1KP common variants: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants"
- [discussion] GIAB truth dataset used for benchmarking: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] Indel calls longer than 4 bp lack exact alternative alleles: "Clairvoyante was set to provide the exact alternative allele only for indel variants ≤4 bp"
- [discussion] Reads realigned to GRCh37 decoy v5 with NGMLR before calling: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
