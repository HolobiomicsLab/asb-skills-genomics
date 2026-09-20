---
name: giab-truth-set-concordance-analysis
description: Use when a called VCF (e.g. from DeepVariant or Clairvoyante on HG001
  Illumina or Oxford Nanopore data) must be scored for precision/recall against a
  GIAB truth set, or when comparing variant callers on identical inputs.
when_to_use_negative:
- Truth sets are unavailable for the chemistry/sample combination — Clairvoyante cannot
  be applied to completely novel sequencing technologies lacking a high-quality GIAB-style
  training dataset
- Evaluating indel calling on ONT data specifically, where indel F1-score remains
  around 50% and the metric is not a meaningful discriminator of caller quality
- The sample under test is not covered by a GIAB high-confidence truth set (e.g. ONT
  data are available only for HG001, limiting cross-sample evaluation)
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Variant caller under benchmark; its calls are scored against the GIAB truth
    set for comparison with DeepVariant
  repo: https://github.com/aquaskyline/Clairvoyante
- name: DeepVariant
  role: Variant caller under benchmark (v0.6.1 with transfer learning); evaluated
    at 0.9995 precision / 0.9991 recall on HG001 Illumina
- name: Clair
  role: Successor of Clairvoyante for germline small variant calling on single-molecule
    sequencing data
  repo: https://github.com/HKU-BAL/Clair
- name: NGMLR
  role: Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 prior to calling
    and concordance evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:6f320e6ba6d1b53344d224bf36796294fe81801d41f08d037e6180707348d614
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
  - Clairvoyante
  - DeepVariant
  - Illumina
  - NGMLR
  - Oxford Nanopore
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# giab-truth-set-concordance-analysis

## Summary

Benchmark a variant caller by evaluating its VCF output against the Genome-in-a-Bottle (GIAB) version 3.3.2 truth set for HG001, computing precision, recall, and F1-score so callers such as DeepVariant and Clairvoyante can be compared on the same data.

## When to use

Use when a called VCF (e.g. from DeepVariant or Clairvoyante on HG001 Illumina or Oxford Nanopore data) must be scored for precision/recall against a GIAB truth set, or when comparing variant callers on identical inputs.

## When NOT to use

- Truth sets are unavailable for the chemistry/sample combination — Clairvoyante cannot be applied to completely novel sequencing technologies lacking a high-quality GIAB-style training dataset
- Evaluating indel calling on ONT data specifically, where indel F1-score remains around 50% and the metric is not a meaningful discriminator of caller quality
- The sample under test is not covered by a GIAB high-confidence truth set (e.g. ONT data are available only for HG001, limiting cross-sample evaluation)

## Inputs

- Variant calls in VCF format from the caller under test (e.g. DeepVariant or Clairvoyante output)
- GIAB version 3.3.2 truth set for HG001
- Sample alignments: HG001 Illumina BAM/CRAM; Oxford Nanopore rel5 alignments for chromosome 1 and chromosome 22
- Reference genome (GRCh37 decoy version 5 for re-aligned SMS reads)

## Outputs

- Per-benchmark precision, recall, and F1-score per caller per dataset/chromosome
- Summary comparison table (e.g. deepvariant_clairvoyante_benchmark.csv)

## How to apply

Run the variant caller on the sample alignments — e.g. DeepVariant v0.6.1 with transfer learning on HG001 Illumina BAMs, or Clairvoyante on Illumina and ONT rel5 alignments — then evaluate the resulting VCF against the GIAB version 3.3.2 HG001 truth set to compute precision, recall, and F1-score. For multi-technology comparisons, keep inputs matched per technology: Illumina datasets were downsampled to 50x coverage to match SMS coverage, and PacBio/ONT reads were re-aligned to GRCh37 decoy version 5 with NGMLR v0.2.3 before calling. Compute per-benchmark metrics for each caller on each dataset/chromosome and record them in a summary table (e.g. deepvariant_clairvoyante_benchmark.csv). Validation is by reproducing reported values within rounding tolerance — e.g. DeepVariant Illumina precision 0.9995 and recall 0.9991, and ONT chr1/chr22 F1-scores of 83.05% and 77.89% against the chr21-trained model. Note that Clairvoyante's own reported baseline on HG001-related benchmarks includes 99.67, 95.78, and 90.53% F1 on 1KP common variants for Illumina, PacBio, and ONT respectively.

## Related tools

- **Clairvoyante** (Variant caller under benchmark; its calls are scored against the GIAB truth set for comparison with DeepVariant) — https://github.com/aquaskyline/Clairvoyante
- **DeepVariant** (Variant caller under benchmark (v0.6.1 with transfer learning); evaluated at 0.9995 precision / 0.9991 recall on HG001 Illumina)
- **Clair** (Successor of Clairvoyante for germline small variant calling on single-molecule sequencing data) — https://github.com/HKU-BAL/Clair
- **NGMLR** (Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 prior to calling and concordance evaluation)

## Examples

```
aws s3 cp s3://nanopore-human-wgs/rel5-nanopore-wgs-chr1.fastq.gz . && clairvoyante.py callVarBamParallel --bam HG001_illumina_50x.bam --ref GRCh37_decoy_v5.fa --out HG001_calls.vcf
```

## Evaluation signals

- Computed Illumina precision/recall match reported DeepVariant values 0.9995 / 0.9991 within rounding tolerance
- ONT F1-scores reproduce 83.05% (chr1) and 77.89% (chr22) against the chr21-transferred model within rounding tolerance
- Same truth set (GIAB v3.3.2 HG001) and same input alignments are used for every caller being compared
- Per-benchmark metrics for all callers are recorded in the summary CSV with no missing dataset/chromosome rows
- Illumina evaluation is performed on 50x-downsampled data to match SMS coverage, so metrics are coverage-comparable

## Limitations

- Clairvoyante's ONT indel F1-score remains around 50%, so whole-genome F1 comparisons mask poor indel performance on ONT data
- Multi-allelic SNPs are excluded from Clairvoyante training and variants with two or more alternative alleles are ignored, biasing concordance for such sites
- Clairvoyante does not consider base quality of sequencing reads; only HG001 ONT data are available, limiting cross-sample evaluation

## Evidence

- [discussion] GIAB truth set version: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [abstract] Clairvoyante F1 baselines: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
- [discussion] Coverage downsampling rationale: "we further downsampled the two datasets to 50× to match the available data coverage of the other two SMS technologies"
- [discussion] SMS realignment step: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] Indel performance caveat: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
