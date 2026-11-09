---
name: snv-indel-precision-recall-evaluation
description: Use when you need to quantify accuracy of germline SNV/indel calls (VCF)
  against a GIAB truth set, e.g. comparing a variant caller's performance on HG001/NA12878
  using genotype-aware TP/FP/FN classification.
when_to_use_negative:
- Evaluating whole-genome performance rather than 1KGP3 common variant sites — the
  paper reports separate whole-genome F1-scores (98.65/92.57/87.26%) and they differ
  substantially from common-site scores
- Indel-heavy ONT benchmarking where the method itself admits limits — the paper notes
  Indel F1-score remains around 50% for ONT data
- Somatic or trio-sample variant calling, which Clairvoyante does not yet support
  and for which this GIAB germline truth-set protocol is not appropriate
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: RTG vcfeval
  role: Genotype-aware benchmarking engine that classifies calls as true positives,
    false positives, and false negatives against the GIAB v3.3.2 HG001 truth dataset
- name: Clairvoyante
  role: Variant caller under test whose VCF output (SNP/indel type, zygosity, alternative
    allele, indel length) is benchmarked by vcfeval
  repo: https://github.com/aquaskyline/Clairvoyante
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:68a29da7bed0b1d1220347e27a5676c19b6e11989ebfb2912fa652b85c03a349
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - RTG vcfeval
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# snv-indel-precision-recall-evaluation

## Summary

Genotype-aware benchmarking of SNV and small-indel variant calls using RTG vcfeval against the GIAB v3.3.2 HG001 truth dataset, computing FPR, FNR, precision, and F1-score at 1KGP3 common variant sites. This is the evaluation scheme the Clairvoyante paper uses to report its best Illumina configuration.

## When to use

Use when you need to quantify accuracy of germline SNV/indel calls (VCF) against a GIAB truth set, e.g. comparing a variant caller's performance on HG001/NA12878 using genotype-aware TP/FP/FN classification.

## When NOT to use

- Evaluating whole-genome performance rather than 1KGP3 common variant sites — the paper reports separate whole-genome F1-scores (98.65/92.57/87.26%) and they differ substantially from common-site scores
- Indel-heavy ONT benchmarking where the method itself admits limits — the paper notes Indel F1-score remains around 50% for ONT data
- Somatic or trio-sample variant calling, which Clairvoyante does not yet support and for which this GIAB germline truth-set protocol is not appropriate

## Inputs

- Called variant VCF (SNPs and small indels) from the caller under test, e.g. Clairvoyante output on 50x HG001 Illumina alignments
- GIAB version 3.3.2 truth VCF/BED for HG001 as RTG vcfeval baseline
- 1KG project phase 3 (1KP) common variant site list for restricting evaluated sites

## Outputs

- vcfeval summary with genotype-aware TP, FP, and FN counts (e.g. vcfeval_summary.txt)
- Derived metrics: FPR, FNR, precision, and F1-score (e.g. 0.25% FPR, 0.41% FNR, 99.75% precision, 99.67% F1 for best Illumina config)

## How to apply

First restrict the caller's emitted genotyped calls (VCF) to the 1KG project phase 3 common variant site list so metrics reflect well-supported regions. Then run RTG vcfeval with the GIAB version 3.3.2 truth dataset for HG001 as the baseline so true positives, false positives, and false negatives are classified genotype-aware rather than position-only. Compute FPR = FP/(FP+TP), FNR = FN/(FN+TP), precision = TP/(TP+FP), and F1-score from the vcfeval summary. Use the article's best Illumina configuration as a reference point: HG001 calls with a model trained on HG001 for 999 epochs yield 0.25% FPR, 0.41% FNR, 99.75% precision, and 99.67% F1-score on 1KP common variants. Save the vcfeval summary (e.g. vcfeval_summary.txt) so precision-recall components can be recomputed and audited. Note the same protocol spans technologies: 99.67, 95.78, and 90.53% F1-score on 1KP common variants for Illumina, PacBio, and Oxford Nanopore respectively.

## Related tools

- **RTG vcfeval** (Genotype-aware benchmarking engine that classifies calls as true positives, false positives, and false negatives against the GIAB v3.3.2 HG001 truth dataset)
- **Clairvoyante** (Variant caller under test whose VCF output (SNP/indel type, zygosity, alternative allele, indel length) is benchmarked by vcfeval) — https://github.com/aquaskyline/Clairvoyante

## Examples

```
rtg vcfeval -b hg001_giab_3.3.2.vcf.gz -c clairvoyante_hg001_1kp.vcf.gz -o vcfeval_out -t SDF && cat vcfeval_out/summary.txt > vcfeval_summary.txt
```

## Evaluation signals

- Precision and F1-score should match the vcfeval summary — for the reference Illumina configuration expect precision 99.75% and F1-score 99.67%
- FPR and FNR derived from vcfeval TP/FP/FN counts should reconcile: reference values are 0.25% FPR and 0.41% FNR
- Evaluated call set must be restricted to the 1KG phase 3 common variant site list; unrestricted metrics will differ from the reported 1KP common-variant F1-scores
- vcfeval must be run genotype-aware against GIAB v3.3.2 HG001; position-only comparison will inflate or deflate TP/FP counts
- Cross-technology sanity check: F1-scores should follow the ordering Illumina (99.67%) > PacBio (95.78%) > ONT (90.53%) on 1KP common variants

## Limitations

- Truth-set dependence: a high-quality GIAB-style dataset must exist for the technology; the paper notes this hinders application to completely novel sequencing technologies and chemistries
- ONP-based indel evaluation is weak — Indel F1-score remains around 50% for ONT data, so indel precision/recall on ONT calls is not reliably benchmarked
- Multi-allelic variants (two or more alternative alleles) are ignored by the evaluated caller, so vcfeval metrics exclude that class of sites

## Evidence

- [abstract] Best Illumina performance reference: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants"
- [discussion] Truth dataset choice: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] Filtering multi-allelic variants: "In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training"
- [readme] Reference accuracy on known variants: "Clairvoyante achieved 99.73%, 97.68% and 95.36% accuracy on known variants"
