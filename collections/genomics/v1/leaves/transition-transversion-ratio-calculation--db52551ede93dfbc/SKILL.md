---
name: transition-transversion-ratio-calculation
description: Use when a filtered variant call set (e.g., Clairvoyante output as VCF/TSV
  with CHROM, POS, REF, ALT) must be typed into SNP/insertion/deletion classes and
  summarized with a Ti/Tv ratio, e.g., profiling concordant PacBio+ONT calls against
  the GIAB truth set.
when_to_use_negative:
- Do not use the resulting Ti/Tv (~1.42 here) as an accuracy certificate — it is a
  candidate/novel-variant catalog metric, not a validated truth-set metric where Ti/Tv
  near 2 is expected
- Not applicable to structural variants or multi-allelic sites — Clairvoyante ignores
  variants with two or more alternative alleles and only reports exact alternative
  alleles for indels <=4 bp
- Not for somatic or trio-based calling — the upstream caller does not support those
  modes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2885
tools:
- name: Clairvoyante
  role: Deep-learning variant caller that produced the PacBio and ONT HG001 call sets
    being classified and summarized with the Ti/Tv ratio
  repo: https://github.com/aquaskyline/Clairvoyante
- name: NGMLR
  role: Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 prior to Clairvoyante
    calling, ensuring the coordinates in the catalog are consistent
- name: GATK UnifiedGenotyper
  role: Reference comparator used to benchmark Clairvoyante's Illumina calls underlying
    the workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:fb8439d9cda2666cc10d5b2d4d9c97bbc586ee06db1ce06e090247d7747b85e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - GATK UnifiedGenotyper
  - NGMLR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transition-transversion-ratio-calculation

## Summary

Classifies a filtered variant catalog into SNPs, insertions, and deletions and computes the transition/transversion (Ti/Tv) ratio of the SNP subset as a sanity metric for candidate variant catalogs, as done for the 3,135-variant HG001 catalog (2,732 SNPs, 298 deletions, 105 insertions, Ti/Tv ~1.42).

## When to use

Use when a filtered variant call set (e.g., Clairvoyante output as VCF/TSV with CHROM, POS, REF, ALT) must be typed into SNP/insertion/deletion classes and summarized with a Ti/Tv ratio, e.g., profiling concordant PacBio+ONT calls against the GIAB truth set.

## When NOT to use

- Do not use the resulting Ti/Tv (~1.42 here) as an accuracy certificate — it is a candidate/novel-variant catalog metric, not a validated truth-set metric where Ti/Tv near 2 is expected
- Not applicable to structural variants or multi-allelic sites — Clairvoyante ignores variants with two or more alternative alleles and only reports exact alternative alleles for indels <=4 bp
- Not for somatic or trio-based calling — the upstream caller does not support those modes

## Inputs

- Genome-wide Clairvoyante HG001 call sets from PacBio and Oxford Nanopore reads at the 0.2 AF cutoff (VCF/TSV with CHROM, POS, REF, ALT, quality)
- GIAB v3.3.2 HG001 truth variant set for subtraction
- PacBio/ONT alignments re-aligned to GRCh37 decoy version 5 with NGMLR v0.2.3 (upstream of variant calling)

## Outputs

- supplementary_data1_variants.tsv — catalog of 3,135 variants (2,732 SNPs, 298 deletions, 105 insertions) absent from GIAB
- variant_summary.txt — per-class counts and Ti/Tv ratio (~1.42)

## How to apply

Start from the concordant call set: intersect Clairvoyante HG001 calls made on PacBio and ONT reads (both generated at the 0.2 allele-fraction cutoff and filtered to geometric-mean variant quality >= 135), then subtract GIAB v3.3.2 HG001 truth variants so only candidates absent from the truth set remain. Classify each retained variant by REF/ALT allele length: single-nucleotide REF and ALT = SNP (2,732 in this catalog), longer ALT than REF = insertion (105), longer REF than ALT = deletion (298). Compute the Ti/Tv ratio on the SNP subset only, counting transitions (A<->G, C<->T) over transversions (A<->C, A<->T, G<->C, G<->T); the expected value here is ~1.42, well below the ~2.0 typical of validated truth sets, reflecting false positives among novel candidates. Write the variant list as supplementary_data1_variants.tsv (Supplementary Data 1 format) and the class counts plus Ti/Tv as variant_summary.txt.

## Related tools

- **Clairvoyante** (Deep-learning variant caller that produced the PacBio and ONT HG001 call sets being classified and summarized with the Ti/Tv ratio) — https://github.com/aquaskyline/Clairvoyante
- **NGMLR** (Re-aligns PacBio and ONT reads to GRCh37 decoy version 5 prior to Clairvoyante calling, ensuring the coordinates in the catalog are consistent)
- **GATK UnifiedGenotyper** (Reference comparator used to benchmark Clairvoyante's Illumina calls underlying the workflow)

## Examples

```
python -c "import pandas as pd; v=pd.read_csv('supplementary_data1_variants.tsv',sep='\t'); s=v[(v.ref.str.len()==1)&(v.alt.str.len()==1)]; ti=(s.ref+s.alt).isin(['AG','GA','CT','TC']).sum(); print(ti/(len(s)-ti))"
```

## Evaluation signals

- Class counts must sum to the catalog size: 2,732 + 298 + 105 = 3,135 variants
- Computed Ti/Tv over the SNP subset should be ~1.42; a value near 0.5 or >2.5 suggests misclassification or a contamination of the SNP set
- Every retained position must be absent from GIAB v3.3.2 HG001 truth variants (truth-set subtraction is lossless on output)
- Each output variant must be concordantly present in both the PacBio and ONT filtered call sets at quality >= 135 (geometric mean)
- Output files conform to the declared schemas: supplementary_data1_variants.tsv (Supplementary Data 1 format) and variant_summary.txt

## Limitations

- Indel detection, especially for ONT data, is weak (Indel F1-score remains around 50%), so the 298 deletions and 105 insertions in the catalog carry high uncertainty
- Clairvoyante ignores variants with two or more alternative alleles and multi-allelic SNPs are excluded from training, so complex sites never reach the catalog
- Exact alternative alleles are only provided for indels <=4 bp and read base quality is not considered, limiting precision of the classified alleles

## Evidence

- [abstract] we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads: "we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [readme] Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type, zygosity, alternative allele and Indel length.: "Clairvoyante, a multi-task five-layer convolutional neural network model for predicting variant type, zygosity, alternative allele and Indel length."
- [discussion] it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
