---
name: vcf-truth-set-benchmarking-rtg-vcfeval
description: Use when a variant-caller output VCF must be scored for accuracy against
  a truth set (e.g., GIAB v3.3.2 HG001) on defined benchmark regions, producing FPR/FNR/precision/F1
  metrics.
when_to_use_negative:
- The sample has no high-quality GIAB-style truth dataset available, as the paper
  notes this hinders applying the pipeline to completely novel sequencing technologies
  and chemistries
- Evaluating variants outside the benchmark regions (e.g., whole-genome rather than
  1KGp3 common-variant stratification) — metrics will not be comparable to region-stratified
  published values
- The VCF contains multi-allelic variants or indels >4 bp requiring exact alternative
  alleles, which the current Clairvoyante design ignores or cannot resolve
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Produces the predicted variant VCF (variant type, zygosity, alternative allele)
    that is benchmarked with RTG vcfeval
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante model performing the
    calls is built
  repo: https://github.com/aquaskyline/Clairvoyante
- name: PacBio
  role: Sequencing technology supplying the 44x HG001 reads whose variant calls are
    benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:af3fc3a5c13aec2b827ff95b56bd97f7be3d5b651eae581eb370f7609b1fcd54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - PacBio
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# vcf-truth-set-benchmarking-rtg-vcfeval

## Summary

Benchmark a caller-generated VCF against a GIAB truth VCF using RTG vcfeval, restricted to matched regions, and derive FPR, FNR, precision, and F1-score from the confusion counts. This is the evaluation protocol the Clairvoyante paper used to report variant-calling accuracy on HG001 across Illumina, PacBio, and Oxford Nanopore data.

## When to use

Use when a variant-caller output VCF must be scored for accuracy against a truth set (e.g., GIAB v3.3.2 HG001) on defined benchmark regions, producing FPR/FNR/precision/F1 metrics.

## When NOT to use

- The sample has no high-quality GIAB-style truth dataset available, as the paper notes this hinders applying the pipeline to completely novel sequencing technologies and chemistries
- Evaluating variants outside the benchmark regions (e.g., whole-genome rather than 1KGp3 common-variant stratification) — metrics will not be comparable to region-stratified published values
- The VCF contains multi-allelic variants or indels >4 bp requiring exact alternative alleles, which the current Clairvoyante design ignores or cannot resolve

## Inputs

- Caller-generated VCF of predicted variants (SNPs and indels ≤4 bp with exact alternative alleles)
- GIAB Genome-in-a-Bottle version 3.3.2 truth VCF for HG001 (NA12878)
- Benchmark region BED/interval list (e.g., 1KGp3 common variant sites)
- Sequencing alignment BAM (e.g., 44x PacBio HG001 realigned to GRCh37 decoy v5 with NGMLR v0.2.3) used to generate the calls

## Outputs

- RTG vcfeval confusion counts (TP/FP/FN)
- Performance metrics table (vcfeval_performance_metrics.tsv) with FPR, FNR, precision, and F1-score

## How to apply

Write the predicted variants (variant type, zygosity, alternative allele) to a VCF file, restricting calls to the benchmark regions (e.g., 1KGp3 common variant sites) and discarding indel calls longer than 4 bp for which the model cannot provide an exact alternative allele. Run RTG vcfeval comparing the predicted VCF against the Genome-in-a-Bottle version 3.3.2 HG001 truth dataset, stratified to the same benchmark regions so that evaluation and prediction cover identical intervals. From the vcfeval confusion counts (true positives, false positives, false negatives), compute false positive rate, false negative rate, precision, and F1-score and write them to a TSV (e.g., vcfeval_performance_metrics.tsv). Verify reproduction by comparing against published values, e.g., the PacBio HG001 best configuration reporting FPR 2.17%, FNR 6.06%, precision 97.70%, F1-score 95.78%.

## Related tools

- **Clairvoyante** (Produces the predicted variant VCF (variant type, zygosity, alternative allele) that is benchmarked with RTG vcfeval) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which the Clairvoyante model performing the calls is built) — https://github.com/aquaskyline/Clairvoyante
- **PacBio** (Sequencing technology supplying the 44x HG001 reads whose variant calls are benchmarked)

## Examples

```
rtg vcfeval --baseline=GIAB_HG001_v3.3.2.vcf.gz --calls=clairvoyante_hg001.vcf.gz --bed-regions=1KGp3_common.bed --output=vcfeval_out
```

## Evaluation signals

- vcfeval confusion counts yield FPR 2.17%, FNR 6.06%, precision 97.70%, F1-score 95.78% for the best PacBio HG001 configuration (HG001-trained 1499-epoch model at 1KGp3 common variant sites)
- Predicted VCF contains no indel calls longer than 4 bp (exact alternative alleles are only provided for indels ≤4 bp)
- vcfeval stratification BED matches the 1KGp3 common-variant regions used for prediction
- Metrics file vcfeval_performance_metrics.tsv is populated with FPR, FNR, precision, and F1-score computed from vcfeval output
- Benchmark truth VCF is GIAB version 3.3.2 for HG001

## Limitations

- Truth-set-based benchmarking depends on GIAB-style high-quality datasets; none exist for many novel sequencing technologies and chemistries
- Region choice matters: 1KGp3 common-variant stratification (95.78% PacBio F1) differs from whole-genome results (92.57% PacBio F1), so metrics are not interchangeable across regions
- Variants with two or more alternative alleles are ignored by the current Clairvoyante design and thus not representable in the benchmarked VCF

## Evidence

- [results] Best PacBio metrics at 1KGp3 common variant sites: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
- [discussion] Truth dataset used for benchmarking: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] Indel length limitation affecting VCF content: "Due to the rareness of long indel variants for model training, Clairvoyante was set to provide the exact alternative allele only for indel variants ≤4 bp"
- [discussion] Benchmarking limits for novel technologies: "This hinders Clairvoyante from being applied to completely novel sequencing technologies and chemistries, for which a high-quality sequencing dataset on standard samples such as GIAB has yet"
- [readme] Clairvoyante open-source repo and purpose: "a multi-task five-layer convolutional neural network model for predicting variant type, zygosity, alternative allele and Indel length"
