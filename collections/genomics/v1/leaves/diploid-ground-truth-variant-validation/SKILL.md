---
name: diploid-ground-truth-variant-validation
description: Use when a diploid assembly or corrected long-read set must be scored
  against ground truth variants — e.g. checking whether error correction retained
  heterozygous SNP alleles or whether haplotypes match curated SVs.
when_to_use_negative:
- Input is PacBio HiFi or Nanopore duplex data — PECAT's pipeline achieves only mediocre
  performance there, and corrected reads plus standard SNP calling is not the appropriate
  validation path
- Evaluation target is low-heterozygosity regions where the article states SNP-call
  accuracy remains low and error correction cannot distinguish small errors from heterozygote
  alleles
- A de novo variant discovery call is needed rather than validation against ground
  truth — this skill assumes truth labels or a curated SV set are available
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Pomoxis
  role: Reference-based evaluation tool used to score assembly/read accuracy against
    truth
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: SV calling on the pair of haplotype assemblies, compared against the curated
    set of SVs
  repo: https://github.com/KolmogorovLab/hapdiff
- name: Clair3
  role: SNP calling on raw Nanopore reads to complement corrected-read SNP calling
    during validation
  repo: https://github.com/HKU-BAL/Clair3
- name: minimap2
  role: Maps corrected reads to first-round assembly ('-x map-pb -c -p 0.5 -r 1000'
    for PacBio CLR) for heterozygous SNP detection
  repo: https://github.com/lh3/minimap2
- name: PECAT
  role: Source of haplotype-aware corrected reads and diploid assemblies under evaluation
  repo: https://github.com/lemene/PECAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:81630429adc31d76835219299b2a2b5a892ab64a577113fd9b33f860961e58f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/pomoxis
  tools:
  - Clair3
  - PECAT
  - Pomoxis
  - hapdiff
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# diploid-ground-truth-variant-validation

## Summary

Validate phased diploid assemblies and corrected long reads by comparing called variants against ground truth: SNPs via reference-based evaluation (Pomoxis) plus SNP calling with Clair3 on raw Nanopore reads, and structural variants via hapdiff against a curated SV set.

## When to use

Use when a diploid assembly or corrected long-read set must be scored against ground truth variants — e.g. checking whether error correction retained heterozygous SNP alleles or whether haplotypes match curated SVs.

## When NOT to use

- Input is PacBio HiFi or Nanopore duplex data — PECAT's pipeline achieves only mediocre performance there, and corrected reads plus standard SNP calling is not the appropriate validation path
- Evaluation target is low-heterozygosity regions where the article states SNP-call accuracy remains low and error correction cannot distinguish small errors from heterozygote alleles
- A de novo variant discovery call is needed rather than validation against ground truth — this skill assumes truth labels or a curated SV set are available

## Inputs

- Simulated diploid long reads (PacBio CLR or Nanopore R9/R10 error profiles) with sequence truth labels
- Corrected reads (e.g. PECAT output corrected_reads.fasta)
- Pair of haplotype-specific assembly contigs
- Curated set of structural variants / ground truth VCF

## Outputs

- SNP allele accuracy table per tool and heterozygosity rate (e.g. snp_allele_accuracy_fig2a.csv)
- VCF of structural variant calls from hapdiff against curated SVs
- Reference-based evaluation accuracy metrics from Pomoxis

## How to apply

Align corrected reads or assembly contigs to a haplotype-collapsed reference using minimap2 with paper-specified parameters ('-x map-pb -c -p 0.5 -r 1000' for PacBio CLR; '-x ava-pb' for PacBio CLR overlaps). Because corrected reads alone are insufficient for accurate SNP calling (unlike Hifiasm with HiFi reads), call SNPs with Clair3 on raw Nanopore reads and additionally combine a corrected-read SNP caller with a raw-read SNP caller; filter candidate SNP sites by coverage in the range [10, 1000] and second-most-common-base thresholds with defaults r_l=0.4, r_h=0.2, c_l=10, c_h=100. Score SNP allele retention by aligning corrected reads against simulated diploid truth and quantifying the fraction of heterozygous SNP alleles retained (PECAT >99% at heterozygosity >=0.0005 vs Canu <80%, MECAT/NECAT <60%). For structural variants, run hapdiff on the two haplotype assemblies and compare calls against the curated SV set. Use Pomoxis as the reference-based evaluation tool for assembly/read accuracy metrics.

## Related tools

- **Pomoxis** (Reference-based evaluation tool used to score assembly/read accuracy against truth) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (SV calling on the pair of haplotype assemblies, compared against the curated set of SVs) — https://github.com/KolmogorovLab/hapdiff
- **Clair3** (SNP calling on raw Nanopore reads to complement corrected-read SNP calling during validation) — https://github.com/HKU-BAL/Clair3
- **minimap2** (Maps corrected reads to first-round assembly ('-x map-pb -c -p 0.5 -r 1000' for PacBio CLR) for heterozygous SNP detection) — https://github.com/lh3/minimap2
- **PECAT** (Source of haplotype-aware corrected reads and diploid assemblies under evaluation) — https://github.com/lemene/PECAT

## Examples

```
minimap2 -x map-pb -c -p 0.5 -r 1000 first_round_assembly.fa corrected_reads.fasta > corrected_vs_asm.sam
```

## Evaluation signals

- PECAT-corrected reads retain heterozygous SNP alleles with >99% accuracy at heterozygosity rates >=0.0005, while Canu is <80% and MECAT/NECAT <60% on the same simulated data
- SNP candidate sites pass the coverage filter in range [10, 1000] before being called heterozygous
- hapdiff SV calls reproduce the curated SV set (hapdiff bundles its own minimap2 version to keep results reproducible across runs)
- Pomoxis reference-based evaluation runs to completion and yields per-contig accuracy metrics for each tool's output
- Validation thresholds mirror Fig. 2a layout when tabulating SNP allele accuracies per tool and heterozygosity rate

## Limitations

- Even after error correction, SNP calling from corrected reads alone is inaccurate for Nanopore-type data, unlike Hifiasm with HiFi reads; raw-read SNP calling (Clair3) must complement it
- SNP call accuracy is still low for low heterozygosity rate regions, and error correction may not distinguish small errors from heterozygote alleles there
- hapdiff results may depend on aligner version and parameters, so it ships a modified minimap2 to ensure reproducibility

## Evidence

- [discussion] We use the reference-based evaluation tool Pomoxis: "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [discussion] SV comparison against curated set: "using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs"
- [discussion] Corrected reads insufficient for SNP calling: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [discussion] minimap2 mapping parameters for corrected reads: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] SNP site coverage filter: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Low-heterozygosity limitation: "Nevertheless, the accuracy of this SNP call is still low for low heterozygosity rate regions."
- [readme] hapdiff bundles minimap2 for reproducibility: [evidence span withheld — anchor status: not_found]
