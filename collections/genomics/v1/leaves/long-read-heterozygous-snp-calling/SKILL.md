---
name: long-read-heterozygous-snp-calling
description: Use when you must identify heterozygous SNP sites from Nanopore/PacBio
  long reads aligned to haplotype-collapsed contigs (e.g., to phase reads or flag
  inconsistent overlaps in a diploid string-graph assembly).
when_to_use_negative:
- Input reads are PacBio HiFi or Nanopore duplex — PECAT achieves only mediocre performance
  there, and Hifiasm-style corrected-read SNP calling suffices for HiFi
- 'Heterozygosity is very low (<0.0005): PECAT''s error correction may not distinguish
  small sequencing errors from heterozygote alleles, so corrected-read SNP calls alone
  are unreliable'
- You already have a trusted diploid de novo assembly and only need SV comparison
  — use hapdiff instead of re-calling SNPs from reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Provides the haplotype-collapsed genome, the corrected-read SNP caller (coverage-range
    and second-most-common-base criteria), and the bisecting k-means read grouping
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Maps corrected/raw reads to first-round contigs for SNP calling (phase_rd2ctg_options,
    e.g., -x map-pb -c -p 0.5 -r 1000)
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Raw-read deep-learning SNP caller used to re-call SNPs for Nanopore reads
    and support the loose 6-allele inconsistent-overlap threshold
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Optional neural-network consensus/variant caller used to further improve assembly
    quality
  repo: https://github.com/nanoporetech/medaka
- name: Pomoxis
  role: Reference-based evaluation utility (also supplies read simulation used to
    benchmark SNP allele accuracy at low heterozygosity)
  repo: https://github.com/nanoporetech/pomoxis
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:850f98b071561c26057997160bf608172f8b23b3648dd32e582ea0d0d40c9d01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - Medaka
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
yes_rate: 0.875
n_claims: 16
---

# long-read-heterozygous-snp-calling

## Summary

Call heterozygous SNP sites from long noisy reads mapped to a haplotype-collapsed assembly, combining a corrected-read SNP caller (base-frequency/second-most-common-base criteria) with a raw-read caller (Clair3 for Nanopore) so SNP alleles can drive inconsistent-overlap identification in PECAT's diploid assembly.

## When to use

Use when you must identify heterozygous SNP sites from Nanopore/PacBio long reads aligned to haplotype-collapsed contigs (e.g., to phase reads or flag inconsistent overlaps in a diploid string-graph assembly).

## When NOT to use

- Input reads are PacBio HiFi or Nanopore duplex — PECAT achieves only mediocre performance there, and Hifiasm-style corrected-read SNP calling suffices for HiFi
- Heterozygosity is very low (<0.0005): PECAT's error correction may not distinguish small sequencing errors from heterozygote alleles, so corrected-read SNP calls alone are unreliable
- You already have a trusted diploid de novo assembly and only need SV comparison — use hapdiff instead of re-calling SNPs from reads

## Inputs

- Corrected long reads (PECAT haplotype-aware correction output, FASTA/FASTQ)
- Haplotype-collapsed first-round contigs ($PROJECT/3-assemble/primary.fasta)
- Raw Nanopore R9/R10 or PacBio CLR reads (FASTQ) for the raw-read SNP caller
- minimap2 alignments of reads to contigs (PAF/SAM with -c for CIGAR)

## Outputs

- List of heterozygous SNP sites passing coverage [10, 1000] and second-most-common-base thresholds
- Per-read SNP-allele vectors H_r(s) over called heterozygous sites
- Haplotype-grouped reads from modified bisecting k-means with verified SNP alleles
- Inconsistent read-pair list (inconsistent_read_pairs.tsv) using 8-allele (corrected) or 6-allele (raw) thresholds

## How to apply

Map reads to the haplotype-collapsed first-round contigs with minimap2 (e.g., '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR; '-x map-ont'-style settings for Nanopore via phase_rd2ctg_options). Retain a site as heterozygous only if (1) its coverage is in the range [10, 1000] and (2) the second-most-common base count passes thresholds r_l=0.4, r_h=0.2, c_l=10, c_h=100. Assign each read its SNP-allele vector over the retained sites and cluster reads by haplotype using PECAT's modified bisecting k-means, which verifies/corrects assigned SNP alleles. Because corrected reads alone do not call SNPs accurately enough (accuracy degrades in low-heterozygosity regions), combine with a raw-read caller: use Clair3 for Nanopore raw reads (loose 6-different-SNP-allele threshold) versus the strict 8-allele threshold on corrected reads, and optionally Medaka/Clair3 to further improve assembly quality.

## Related tools

- **PECAT** (Provides the haplotype-collapsed genome, the corrected-read SNP caller (coverage-range and second-most-common-base criteria), and the bisecting k-means read grouping) — https://github.com/lemene/PECAT
- **minimap2** (Maps corrected/raw reads to first-round contigs for SNP calling (phase_rd2ctg_options, e.g., -x map-pb -c -p 0.5 -r 1000)) — https://github.com/lh3/minimap2
- **Clair3** (Raw-read deep-learning SNP caller used to re-call SNPs for Nanopore reads and support the loose 6-allele inconsistent-overlap threshold) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Optional neural-network consensus/variant caller used to further improve assembly quality) — https://github.com/nanoporetech/medaka
- **Pomoxis** (Reference-based evaluation utility (also supplies read simulation used to benchmark SNP allele accuracy at low heterozygosity)) — https://github.com/nanoporetech/pomoxis

## Examples

```
minimap2 -x map-pb -c -p 0.5 -r 1000 $PROJECT/3-assemble/primary.fasta corrected_reads.fq > reads2ctg.paf
```

## Evaluation signals

- Every called SNP site has coverage within [10, 1000] and second-most-common-base counts satisfying r_l=0.4, r_h=0.2, c_l=10, c_h=100
- Each flagged inconsistent read pair contains at least 8 different SNP alleles (corrected reads) or at least 6 different SNP alleles (raw reads)
- Round-2 inconsistent-overlap percentage in the simplified string graph drops sharply below round-1 values (paper: from ~16-37% down to 0.03-1.09% across seven diploid datasets)
- SNP allele accuracy against simulated truth variants (e.g., Pomoxis-simulated diploid genomes at heterozygosity 0.0001) should reach ~96-98% for raw reads; corrected-read-only calls dropping to 92-94% signal the known low-heterozygosity failure mode
- SNP calling is combined with Clair3 re-calls for Nanopore datasets, since corrected reads alone are insufficient for accurate SNP calling

## Limitations

- Corrected reads alone cannot accurately call SNPs, unlike Hifiasm with HiFi reads; a raw-read caller (Clair3 for Nanopore) must supplement them
- SNP call accuracy is still low for low-heterozygosity-rate regions (e.g., at heterozygosity 0.0001, corrected-read SNP allele accuracy drops to 92-94%, below raw reads at 96-98%)
- PECAT's error correction may fail to distinguish small errors from heterozygote alleles in very low heterozygosity regions, even with highly accurate reads
- PECAT needs higher data coverage to obtain a high-quality phased assembly

## Evidence

- [discussion] Coverage range for heterozygous SNP retention: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Second-most-common-base thresholds: "(2) The number of second-most common bases is greater or equal to"
- [discussion] minimap2 mapping parameters for SNP calling: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] Rationale for combining raw-read SNP caller: "Therefore, we use Clair3 to call SNPs again for Nanopore reads."
- [discussion] Corrected-read SNP insufficiency: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [discussion] Low-heterozygosity failure mode: "the accuracy of this SNP call is still low for low heterozygosity rate regions."
- [discussion] Bisecting k-means read grouping: "we use a modified bisecting k-means algorithm49 to divide the group"
