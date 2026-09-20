---
name: precision-recall-f1-computation
description: Use when benchmarking a variant caller's VCF output against a GIAB HG001
  truth set and you need whole-genome or per-chromosome precision, recall, and F1-score
  to compare callers or verify reported metrics.
when_to_use_negative:
- You are calling somatic variants or need trio-based analysis — Clairvoyante's evaluation
  pipeline here targets germline small variants only against a GIAB truth set
- The callset includes multi-allelic SNPs or indels >4 bp — these are excluded or
  only approximately typed in the Clairvoyante design, so F1 against them is not meaningful
- You need performance on a completely novel sequencing technology/chemistry for which
  no GIAB-style training/truth dataset exists
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: RTG vcfeval
  role: Haplotype-aware genotype matching of caller VCF against the GIAB HG001 truth
    set, yielding TP/FP/FN counts
- name: Clairvoyante
  role: TensorFlow five-layer CNN variant caller whose VCFs are benchmarked; run with
    0.2 alternative allele frequency cutoff for genome-wide calls
  repo: https://github.com/aquaskyline/Clairvoyante
- name: DeepVariant
  role: Baseline caller (v0.6.1 with transfer learning) whose Illumina precision/recall
    and ONT per-chromosome F1-scores are verified against reported values
- name: NGMLR
  role: Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) prior to
    calling, fixing the evaluation input
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:fb1e9fbeff7b85469954073557780d949641d589111e10287670c4a7d6e4f5ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - DeepVariant
  - Illumina
  - NGMLR
  - Oxford Nanopore
  - PacBio
  - RTG vcfeval
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# precision-recall-f1-computation

## Summary

Compute precision, recall, and F1-score for germline variant callsets by comparing caller VCFs against the Genome-in-a-Bottle v3.3.2 HG001 truth dataset with RTG vcfeval's haplotype-aware genotype matching, then extracting TP/FP/FN counts from the vcfeval output. This is the standard benchmarking step for evaluating variant callers such as Clairvoyante and DeepVariant across Illumina, PacBio, and Oxford Nanopore data.

## When to use

Use when benchmarking a variant caller's VCF output against a GIAB HG001 truth set and you need whole-genome or per-chromosome precision, recall, and F1-score to compare callers or verify reported metrics.

## When NOT to use

- You are calling somatic variants or need trio-based analysis — Clairvoyante's evaluation pipeline here targets germline small variants only against a GIAB truth set
- The callset includes multi-allelic SNPs or indels >4 bp — these are excluded or only approximately typed in the Clairvoyante design, so F1 against them is not meaningful
- You need performance on a completely novel sequencing technology/chemistry for which no GIAB-style training/truth dataset exists

## Inputs

- Caller-produced VCF (e.g., Clairvoyante output from callVarBam.py, or DeepVariant v0.6.1 VCF)
- Genome-in-a-Bottle (GIAB) dataset version 3.3.2 HG001 truth VCF
- Reference genome alignment (GRCh37 decoy version 5 for PacBio/ONT via NGMLR v0.2.3; Illumina downsampled to 50x)

## Outputs

- RTG vcfeval output directory containing TP, FP, and FN variant counts
- Summary CSV of per-technology or per-chromosome precision, recall, and F1-score (e.g., vcfeval_f1_scores.csv, deepvariant_clairvoyante_benchmark.csv)

## How to apply

Run the caller (e.g., Clairvoyante genome-wide with the 0.2 alternative allele frequency cutoff, or DeepVariant v0.6.1 with transfer learning) on the target BAM to produce a VCF. Compare the callset VCF against the Genome-in-a-Bottle version 3.3.2 HG001 truth dataset using RTG vcfeval with haplotype-aware genotype matching. Extract true-positive, false-positive, and false-negative counts from the vcfeval output and compute Precision = TP/(TP+FP), Recall = TP/(TP+FN), and F1 = 2*P*R/(P+R) per technology or per chromosome. Record the metrics in a summary table (e.g., vcfeval_f1_scores.csv or deepvariant_clairvoyante_benchmark.csv) and validate against reported values within rounding tolerance — e.g., Clairvoyante whole-genome F1 of 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore rel5), or DeepVariant v0.6.1 Illumina precision 0.9995 and recall 0.9991. Note that metrics depend on preprocessing choices, so keep the alignment and downsampling steps fixed: Illumina downsampled to 50×, PacBio/ONT realigned to GRCh37 decoy version 5 with NGMLR v0.2.3, ONT rel3 MinION 1D R9.4 reads.

## Related tools

- **RTG vcfeval** (Haplotype-aware genotype matching of caller VCF against the GIAB HG001 truth set, yielding TP/FP/FN counts)
- **Clairvoyante** (TensorFlow five-layer CNN variant caller whose VCFs are benchmarked; run with 0.2 alternative allele frequency cutoff for genome-wide calls) — https://github.com/aquaskyline/Clairvoyante
- **DeepVariant** (Baseline caller (v0.6.1 with transfer learning) whose Illumina precision/recall and ONT per-chromosome F1-scores are verified against reported values)
- **NGMLR** (Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) prior to calling, fixing the evaluation input)

## Examples

```
rtg vcfeval --baseline giab_hg001_v3.3.2.vcf.gz --calls clairvoyante_wgs.vcf.gz --output eval_dir --vcfeval-path rtg
```

## Evaluation signals

- Computed whole-genome F1-scores must match reported values within rounding tolerance: 98.65% (Illumina), 92.57% (PacBio), 87.26% (Oxford Nanopore)
- DeepVariant v0.6.1 Illumina metrics must reproduce precision 0.9995 and recall 0.9991 on HG001 against GIAB v3.3.2
- DeepVariant ONT F1-scores must match 83.05% (chr1) and 77.89% (chr22) using the chr21-trained transferred model
- Precision, recall, and F1 must satisfy F1 = 2*P*R/(P+R) exactly given the extracted vcfeval TP/FP/FN counts
- vcfeval must be run in haplotype-aware genotype matching mode, not naive position matching, or counts will not reproduce

## Limitations

- Indel detection, especially for ONT data, remains poor — Indel F1-score is around 50%, so overall F1 conflates strong SNP performance with weak indel performance
- Only HG001 ONT data were available, limiting cross-sample evaluation and combined-sample training on ONT
- Reported F1 values depend on fixed preprocessing (Illumina downsampled to 50x, realignment to GRCh37 decoy v5 with NGMLR 0.2.3); changing these shifts the metrics

## Evidence

- [other] Compare each technology's VCF against the Genome-in-a-Bottle v3.3.2 HG001 truth dataset using RTG vcfeval for haplotype-aware genotype matching.: [evidence span withheld — anchor status: not_found]
- [abstract] Whole-genome F1-scores per technology: "98.65, 92.57, 87.26% F1-score for whole-genome analysis, using Illumina, PacBio, and Oxford Nanopore data, respectively"
- [discussion] Truth dataset choice: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] Preprocessing that fixes evaluation inputs: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] Indel F1 weakness on ONT: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
