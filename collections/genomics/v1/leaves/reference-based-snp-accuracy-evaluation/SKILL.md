---
name: reference-based-snp-accuracy-evaluation
description: Use when you need to quantify per-allele SNP accuracy of corrected long
  reads (e.g., PECAT output at heterozygosity < 0.0005) against a known truth set,
  or decide whether raw-read callers must supplement corrected-read calling.
when_to_use_negative:
- When reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves
  mediocre performance on these, so corrected-read SNP evaluation is not the relevant
  benchmark
- When evaluating assembly contiguity (phase block NG50) rather than SNP allele retention
  — use assembly QA (e.g., Pomoxis reference-based evaluation of contigs) instead
- When no truth set or simulated diploid reference exists — per-allele accuracy cannot
  be computed without known truth variants
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Haplotype-aware error correction (`pecat.pl correct`) and corrected-read SNP
    calling against the haplotype-collapsed genome using coverage [10, 1000] and second-most-common-base
    thresholds
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Aligns corrected reads to the haplotype-collapsed genome (e.g., '-x map-pb
    -c -p 0.5 -r 1000' for PacBio CLR) prior to SNP calling
  repo: https://github.com/lh3/minimap2
- name: Pomoxis
  role: Reference-based simulation of diploid truth genomes and evaluation; source
    of simulated truth variants for per-allele accuracy
  repo: https://github.com/nanoporetech/pomoxis
- name: Clair3
  role: Cross-check SNP calls on raw Nanopore reads, since corrected reads alone cannot
    accurately call SNPs
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:066d3694359f33bbd4796005a8c7cb487eb6f41aed89bd4242c57c57404f9685
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - PECAT
  - Pomoxis
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-based-snp-accuracy-evaluation

## Summary

Evaluates SNP allele accuracy of corrected/raw long reads by aligning them to a haplotype-collapsed reference with minimap2, calling heterozygous SNPs, and comparing against simulated truth variants (e.g., Pomoxis-generated diploid reference at heterozygosity 0.0001) to quantify how well error correction preserves heterozygous alleles.

## When to use

Use when you need to quantify per-allele SNP accuracy of corrected long reads (e.g., PECAT output at heterozygosity < 0.0005) against a known truth set, or decide whether raw-read callers must supplement corrected-read calling.

## When NOT to use

- When reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves mediocre performance on these, so corrected-read SNP evaluation is not the relevant benchmark
- When evaluating assembly contiguity (phase block NG50) rather than SNP allele retention — use assembly QA (e.g., Pomoxis reference-based evaluation of contigs) instead
- When no truth set or simulated diploid reference exists — per-allele accuracy cannot be computed without known truth variants

## Inputs

- Corrected long reads (e.g., PECAT `pecat.pl correct` output, FASTA/FASTQ)
- Raw long noisy reads (Nanopore R9/R10 or PacBio CLR, FASTQ)
- Simulated diploid reference genome with known heterozygous SNPs (e.g., Pomoxis-generated, heterozygosity 0.0001)
- Haplotype-collapsed reference genome for read mapping
- Truth variant set from the simulated diploid reference

## Outputs

- Per-allele SNP accuracy table (e.g., snp_accuracy_0.0001.csv)
- Called heterozygous SNP set from corrected reads (coverage in [10, 1000], second-most-common-base criteria)
- Optional raw-read SNP calls from Clair3 for comparison

## How to apply

Simulate a diploid reference with known heterozygous SNPs (e.g., Pomoxis reference-based simulation at heterozygosity 0.0001) and generate long noisy Nanopore-like reads from it. After error correction (e.g., PECAT with r_l/r_h/c_l/c_h defaults 0.4/0.2/10/100), align corrected reads to the haplotype-collapsed genome with minimap2 and call heterozygous SNPs using PECAT's coverage-range [10, 1000] and second-most-common-base criteria. Optionally cross-check with Clair3 on raw Nanopore reads, as the paper does for Nanopore data. Compare called SNP alleles against simulated truth variants per allele and write results (e.g., snp_accuracy_0.0001.csv). Acceptance benchmark: corrected-read SNP accuracy should at least match raw-read accuracy (~96-98%); at heterozygosity 0.0001 unmodified PECAT correction drops to 92-94%, below raw reads, indicating the correction alone fails to retain heterozygous SNPs.

## Related tools

- **PECAT** (Haplotype-aware error correction (`pecat.pl correct`) and corrected-read SNP calling against the haplotype-collapsed genome using coverage [10, 1000] and second-most-common-base thresholds) — https://github.com/lemene/PECAT
- **minimap2** (Aligns corrected reads to the haplotype-collapsed genome (e.g., '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR) prior to SNP calling) — https://github.com/lh3/minimap2
- **Pomoxis** (Reference-based simulation of diploid truth genomes and evaluation; source of simulated truth variants for per-allele accuracy) — https://github.com/nanoporetech/pomoxis
- **Clair3** (Cross-check SNP calls on raw Nanopore reads, since corrected reads alone cannot accurately call SNPs) — https://github.com/HKU-BAL/Clair3

## Examples

```
minimap2 -ax map-ont haplotype-collapsed-genome.fa corrected_reads.fq > corrected.sam  # then call SNPs and compare vs Pomoxis truth
```

## Evaluation signals

- Raw-read SNP allele accuracy is ~96-98%; corrected-read accuracy falling below this (e.g., 92-94% at heterozygosity 0.0001) signals failed heterozygote retention
- Every called heterozygous SNP site must satisfy PECAT's filters: coverage in range [10, 1000] and second-most-common-base count above the r_l/r_h/c_l/c_h (0.4/0.2/10/100) thresholds
- Per-allele accuracy is computed by exact matching of called SNP alleles against the simulated truth variants (SNVs from the Pomoxis diploid reference)
- Results are recorded per heterozygosity condition (e.g., snp_accuracy_0.0001.csv) so the low-heterozygosity degradation can be compared across conditions

## Limitations

- At heterozygosity 0.0001, PECAT-corrected reads yield 92-94% SNP allele accuracy, below raw reads (96-98%); 'the accuracy of this SNP call is still low for low heterozygosity rate regions'
- The error correction 'may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions'
- Corrected reads alone cannot accurately call SNPs even after error correction, unlike Hifiasm with HiFi reads; raw-read callers such as Clair3 must supplement

## Evidence

- [discussion] even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads.: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [discussion] Nevertheless, the accuracy of this SNP call is still low for low heterozygosity rate regions.: "Nevertheless, the accuracy of this SNP call is still low for low heterozygosity rate regions."
- [discussion] We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis): "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [discussion] We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] (1) Its coverage is in the range [10, 1000].: "(1) Its coverage is in the range [10, 1000]."
