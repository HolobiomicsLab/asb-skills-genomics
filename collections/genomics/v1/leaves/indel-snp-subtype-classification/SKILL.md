---
name: indel-snp-subtype-classification
description: Use when a concordant filtered variant list (e.g. PacBio+ONT calls absent
  from GIAB) must be decomposed into SNP/insertion/deletion counts and quality-checked
  via its Ti/Tv ratio.
when_to_use_negative:
- The input call set still contains multi-allelic SNPs or multi-allelic sites — Clairvoyante's
  design ignores variants with two or more alternative alleles, so subtype counts
  would be biased
- The variant list has not yet been intersected across technologies and subtracted
  against GIAB truth — classification of the raw concordant calls answers a different
  question
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Multi-task convolutional neural network that predicts variant type (SNP or
    indel), zygosity, alternative allele, and indel length, providing the subtype
    labels for classification
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Clair
  role: Successor variant caller to Clairvoyante for germline small variant calling
    from single-molecule sequencing data; alternate caller whose output can be classified
    the same way
  repo: https://github.com/HKU-BAL/Clair
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:e1b2ba1c6c209320992a071d06e5718a10b036a459a61b5c24d9b910f791bdf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
  - Clairvoyante
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# indel-snp-subtype-classification

## Summary

Classifies a filtered variant call set into SNP, insertion, and deletion subtypes and computes the transition/transversion (Ti/Tv) ratio, as used to characterize the 3,135-variant HG001 catalog (2,732 SNPs, 298 deletions, 105 insertions, Ti/Tv ~1.42) absent from the GIAB v3.3.2 truth set.

## When to use

Use when a concordant filtered variant list (e.g. PacBio+ONT calls absent from GIAB) must be decomposed into SNP/insertion/deletion counts and quality-checked via its Ti/Tv ratio.

## When NOT to use

- The input call set still contains multi-allelic SNPs or multi-allelic sites — Clairvoyante's design ignores variants with two or more alternative alleles, so subtype counts would be biased
- The variant list has not yet been intersected across technologies and subtracted against GIAB truth — classification of the raw concordant calls answers a different question

## Inputs

- Filtered, concordant PacBio+ONT Clairvoyante VCF/call set with GIAB v3.3.2 HG001 truth variants removed
- Per-variant records carrying variant type (SNP or indel), zygosity, alternative allele, and indel length as predicted by Clairvoyante

## Outputs

- supplementary_data1_variants.tsv — per-variant list with subtype annotation
- variant_summary.txt — subtype counts (SNPs, insertions, deletions) and Ti/Tv ratio of the catalog

## How to apply

After intersecting quality-filtered PacBio and ONT Clairvoyante call sets (geometric-mean variant quality >= 135 at the 0.2 AF cutoff) and subtracting GIAB v3.3.2 HG001 truth variants, classify each retained record by variant type: SNPs (single-nucleotide substitution), insertions, and deletions as predicted by Clairvoyante's multi-task output head. Report per-subtype counts (expected format: 2,732 SNPs, 298 deletions, 105 insertions for the HG001 catalog). Compute the Ti/Tv ratio by tallying purine-pyrimidine transitions (A<->G, C<->T) versus transversions among SNPs; the catalog's ~1.42 Ti/Tv is the sanity benchmark, lower than the ~2 expected genome-wide because novel candidates are enriched for spurious or hard-to-map sites. Classify and count SNPs, insertions, and deletions, then compute the transition/transversion ratio of the catalog, writing per-variant records to supplementary_data1_variants.tsv and summary counts to variant_summary.txt.

## Related tools

- **Clairvoyante** (Multi-task convolutional neural network that predicts variant type (SNP or indel), zygosity, alternative allele, and indel length, providing the subtype labels for classification) — https://github.com/aquaskyline/Clairvoyante
- **Clair** (Successor variant caller to Clairvoyante for germline small variant calling from single-molecule sequencing data; alternate caller whose output can be classified the same way) — https://github.com/HKU-BAL/Clair

## Evaluation signals

- Subtype counts sum to the total catalog size: 2,732 + 298 + 105 = 3,135 variants
- Computed Ti/Tv ratio of the SNP subset falls near ~1.42 for the HG001 GIAB-absent catalog
- Every classified record is absent from the GIAB v3.3.2 HG001 truth set
- Each output record carries exactly one of the three subtype labels (SNP, insertion, deletion) consistent with Clairvoyante's predicted variant type and indel length

## Limitations

- Clairvoyante provides the exact alternative allele only for indel variants <=4 bp due to scarcity of long indel training samples, so longer indels may be incompletely classified
- The current design of Clairvoyante ignores variants with two or more alternative alleles, so multi-allelic sites are excluded from the classified catalog
- Indel detection, especially for ONT data, needs substantial improvement (Indel F1-score remains around 50%), so deletion/insertion counts carry higher uncertainty than SNP counts
- Base quality of sequencing reads is not considered in the current Clairvoyante implementation

## Evidence

- [abstract] we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads: "we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads"
- [abstract] a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length: "we developed Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type (SNP or indel), zygosity, alternative allele and indel length"
- [discussion] In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training: "In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training"
- [discussion] it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
