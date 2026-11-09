---
name: read-haplotype-clustering-bisecting-kmeans
description: Use when phasing long noisy reads (PacBio CLR, Nanopore R9/R10) against
  haplotype-collapsed contigs to partition reads by haplotype and flag inconsistent
  overlaps for a second-round diploid string graph assembly.
when_to_use_negative:
- Data is PacBio HiFi or Nanopore duplex — PECAT achieves mediocre phased-assembly
  performance on these high-accuracy read types
- Input is only raw reads without corrected reads or a first-round haplotype-collapsed
  assembly — the SNP-allele vectors require corrected reads aligned to collapsed contigs
- Genomic regions with very low heterozygosity rate — SNP calling accuracy is too
  low to support reliable haplotype clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Implements the modified bisecting k-means read grouping and SNP-allele verification
    as part of its two-round diploid assembly pipeline
  repo: github.com/lemene/PECAT
- name: minimap2
  role: Maps corrected reads to haplotype-collapsed first-round contigs to obtain
    base frequencies for SNP calling
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Optionally re-calls SNPs on raw Nanopore reads because corrected reads alone
    are insufficient for accurate SNP calling
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:05969ebfa124174dcd5e6a930bf9dccf1496bc14e9f2a4e59b226a358562c48f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# read-haplotype-clustering-bisecting-kmeans

## Summary

PECAT's divide-then-combine bisecting k-means method clusters long reads into haplotype groups using SNP-allele vectors extracted from corrected reads aligned to haplotype-collapsed first-round contigs, verifying and correcting assigned SNP alleles to identify inconsistent overlaps.

## When to use

Use when phasing long noisy reads (PacBio CLR, Nanopore R9/R10) against haplotype-collapsed contigs to partition reads by haplotype and flag inconsistent overlaps for a second-round diploid string graph assembly.

## When NOT to use

- Data is PacBio HiFi or Nanopore duplex — PECAT achieves mediocre phased-assembly performance on these high-accuracy read types
- Input is only raw reads without corrected reads or a first-round haplotype-collapsed assembly — the SNP-allele vectors require corrected reads aligned to collapsed contigs
- Genomic regions with very low heterozygosity rate — SNP calling accuracy is too low to support reliable haplotype clustering

## Inputs

- Corrected long reads from PECAT's haplotype-aware error correction step
- Haplotype-collapsed first-round contigs ($PROJECT/3-assemble/primary.fasta)
- Read-to-contig alignments (minimap2 PAF/SAM with -c)
- Raw long reads (for loose-threshold inconsistency checks and Clair3 SNP calling on Nanopore data)

## Outputs

- Haplotype group assignments per read
- Per-read SNP-allele vectors H_r(s) over called heterozygous sites
- Verified/corrected SNP alleles per read
- Flagged inconsistent read pairs (inconsistent_read_pairs.tsv)

## How to apply

Map corrected reads to the haplotype-collapsed first-round contigs with minimap2 (e.g. '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR). Call heterozygous SNP sites from alignment base frequencies, retaining sites with coverage in [10, 1000] whose second-most-common base counts pass thresholds r_l=0.4, r_h=0.2, c_l=10, c_h=100. Assign each read a SNP-allele vector H_r(s) over the called sites, then cluster reads with the modified bisecting k-means algorithm (default parameters p0..p9) to assign reads to haplotype groups and verify/correct the assigned SNP alleles. For Nanopore reads, SNP calls can be re-derived with Clair3 since corrected reads alone are insufficient for accurate SNP calling. The clustered haplotype assignments feed into inconsistency detection: read pairs with >=8 different SNP alleles (corrected reads) or >=6 (raw reads) are marked inconsistent and removed before the second-round string graph assembly.

## Related tools

- **PECAT** (Implements the modified bisecting k-means read grouping and SNP-allele verification as part of its two-round diploid assembly pipeline) — github.com/lemene/PECAT
- **minimap2** (Maps corrected reads to haplotype-collapsed first-round contigs to obtain base frequencies for SNP calling) — https://github.com/lh3/minimap2
- **Clair3** (Optionally re-calls SNPs on raw Nanopore reads because corrected reads alone are insufficient for accurate SNP calling) — https://github.com/HKU-BAL/Clair3

## Examples

```
minimap2 -x map-pb -c -p 0.5 -r 1000 $PROJECT/3-assemble/primary.fasta corrected_reads.fasta > reads_rd2ctg.paf
```

## Evaluation signals

- inconsistent_read_pairs.tsv exists and is parseable
- Every flagged read pair satisfies its applicable allele-count threshold (>=8 different SNP alleles for corrected reads, >=6 for raw reads)
- SNP sites retained for clustering have coverage within [10, 1000]
- Retained heterozygous SNP sites meet the second-most-common-base thresholds (r_l=0.4, r_h=0.2, c_l=10, c_h=100)
- Overlap direction/distance for flagged pairs is consistent with the contigs within the default distance difference of less than 1000 (raw-read threshold max(2500, 0.05*Dc))

## Limitations

- SNP calling accuracy is low in low heterozygosity rate regions
- Corrected reads alone are insufficient for accurate SNP calling — for Nanopore reads Clair3 must be used to call SNPs again
- PECAT needs higher data coverage to obtain a high-quality phased assembly

## Evidence

- [discussion] We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] modified bisecting k-means to verify/correct SNP alleles: "we use a modified bisecting k-means algorithm49 to divide the group"
- [discussion] (1) Its coverage is in the range [10, 1000].: "(1) Its coverage is in the range [10, 1000]."
- [discussion] strict threshold that the pair of reads should contain 8 different SNP alleles: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads.: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
