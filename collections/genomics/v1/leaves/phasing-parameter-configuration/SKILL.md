---
name: phasing-parameter-configuration
description: Use when reconstructing a diploid assembly from long noisy reads (PacBio
  CLR, Nanopore R9/R10) and you need to parameterize the read-phasing step that removes
  inconsistent overlaps between haplotypes.
when_to_use_negative:
- Input reads are PacBio HiFi or Nanopore duplex — PECAT's phasing/correction is tuned
  for long noisy reads and achieves mediocre performance on these data types
- The organism/region has very low heterozygosity — SNP calling accuracy is low there
  and error correction cannot separate small errors from heterozygote alleles
- Dataset coverage is too low — the [10, 1000] coverage filter on SNP sites requires
  sufficient depth and PECAT needs higher coverage for high-quality phased assembly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: PECAT
  role: Implements the haplotype-collapsed assembly, read-level SNP caller, bisecting
    k-means grouping, and inconsistent-overlap flagging whose parameters this skill
    configures
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Maps corrected reads to first-round contigs with '-x map-pb -c -p 0.5 -r 1000'
    (PacBio CLR) to supply base frequencies for SNP calling
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Optional raw-read SNP caller used to call SNPs again for Nanopore reads (v0.1-r12+)
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:260d696a26f1241bde66909bb2421fdda983d1bab1b0b9325e219ffd7f6b626e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# phasing-parameter-configuration

## Summary

Configure the PECAT second-round phasing parameters — heterozygous SNP-caller thresholds (coverage [10,1000]; r_l=0.4, r_h=0.2, c_l=10, c_h=100), strict/loose allele-count cutoffs (8 vs 6), and overlap distance checks — so inconsistent overlaps between haplotypes are reliably flagged before string graph assembly.

## When to use

Use when reconstructing a diploid assembly from long noisy reads (PacBio CLR, Nanopore R9/R10) and you need to parameterize the read-phasing step that removes inconsistent overlaps between haplotypes.

## When NOT to use

- Input reads are PacBio HiFi or Nanopore duplex — PECAT's phasing/correction is tuned for long noisy reads and achieves mediocre performance on these data types
- The organism/region has very low heterozygosity — SNP calling accuracy is low there and error correction cannot separate small errors from heterozygote alleles
- Dataset coverage is too low — the [10, 1000] coverage filter on SNP sites requires sufficient depth and PECAT needs higher coverage for high-quality phased assembly

## Inputs

- Haplotype-collapsed first-round assembly contigs (e.g. $PROJECT/3-assemble/primary.fasta)
- Corrected long reads (FASTA/FASTQ) from PECAT haplotype-aware error correction
- Raw long reads for the loose raw-read SNP caller (e.g. Clair3 for Nanopore reads)
- Read-to-contig alignments from minimap2 (PAF/SAM)

## Outputs

- Heterozygous SNP sites passing coverage and base-count filters
- Per-read SNP-allele vectors H_r(s) and haplotype groups from bisecting k-means
- inconsistent_read_pairs.tsv listing read pairs flagged as inconsistent overlaps

## How to apply

Map corrected reads to the haplotype-collapsed first-round contigs with minimap2 using phase_rd2ctg_options (e.g. '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR). Call heterozygous SNP sites retaining only sites whose coverage is in [10, 1000] and whose second-most-common base counts pass r_l=0.4, r_h=0.2, c_l=10, c_h=100. Assign each read its SNP-allele vector over called sites, then cluster reads with the divide-then-combine modified bisecting k-means algorithm (parameters p0..p9) to group reads by haplotype and verify/correct assigned SNP alleles. Flag inconsistent read pairs with the strict threshold of 8 different SNP alleles for corrected reads (loose 6 for raw reads), additionally checking overlap direction/distance against the contigs with distance difference < 1000 and a raw-read distance threshold of max(2500, 0.05*Dc).

## Related tools

- **PECAT** (Implements the haplotype-collapsed assembly, read-level SNP caller, bisecting k-means grouping, and inconsistent-overlap flagging whose parameters this skill configures) — https://github.com/lemene/PECAT
- **minimap2** (Maps corrected reads to first-round contigs with '-x map-pb -c -p 0.5 -r 1000' (PacBio CLR) to supply base frequencies for SNP calling) — https://github.com/lh3/minimap2
- **Clair3** (Optional raw-read SNP caller used to call SNPs again for Nanopore reads (v0.1-r12+)) — https://github.com/HKU-BAL/Clair3

## Examples

```
minimap2 -x map-pb -c -p 0.5 -r 1000 primary.fasta corrected_reads.fastq > rd2ctg.paf
```

## Evaluation signals

- inconsistent_read_pairs.tsv exists and is parseable as a TSV of read pairs
- Every flagged read pair satisfies its applicable allele-count threshold: >= 8 different SNP alleles for corrected reads or >= 6 for raw reads
- All retained heterozygous SNP sites have coverage within [10, 1000]
- Every retained SNP site's second-most-common base count passes r_l=0.4, r_h=0.2, c_l=10, c_h=100
- All flagged pairs' overlap direction/distance agree with the contigs within the < 1000 distance difference (and max(2500, 0.05*Dc) raw-read threshold)

## Limitations

- SNP calling accuracy from corrected reads is low in low heterozygosity rate regions
- Error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions
- Assembly parameters used in the study are only described in Supplementary Notes 4 and 5, not the main text, so exact defaults for p0..p9 may require consulting the supplement
- PECAT needs higher data coverage to obtain high-quality phased assemblies

## Evidence

- [discussion] SNP site coverage filter: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Strict allele threshold for corrected reads: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Distance check against contigs: "The difference between two distances should be less than 1000 by default."
- [discussion] Bisecting k-means grouping of reads: "we use a modified bisecting k-means algorithm49 to divide the group"
- [readme] minimap2 mapping invocation for PacBio CLR: [evidence span withheld — anchor status: external_doc]
