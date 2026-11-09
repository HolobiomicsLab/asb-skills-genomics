---
name: vcf-variant-table-filtering
description: Use when candidate SNVs from error-prone long-read alignments (BAM/CRAM
  of PacBio SMRT or Oxford Nanopore reads) must be filtered for strand bias before
  haplotype assembly or genotyping.
when_to_use_negative:
- Input variants are already filtered final genotype calls with strand-bias annotations
  — re-filtering may remove true positives without benefit
- The data are short reads (Illumina) from tools like GATK HapCUT2Caller or FreeBayes,
  where the long-read strand-bias error model of Longshot does not apply
- Calling structural variants or indels de novo — Longshot only calls SNVs (indels
  can be genotyped only if supplied in an input VCF)
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Implements the strand-bias Fisher's exact filter (--strand_bias_pvalue_cutoff,
    two-tailed p < 0.01) on candidate SNVs prior to haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:63b59c458529939d9e43b91ba950ea0035d4f9f822194b876710e9c85da951a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# vcf-variant-table-filtering

## Summary

Filters candidate SNV calls in a VCF using statistical and coverage-based criteria — most notably a two-tailed Fisher's exact test for strand bias (p < 0.01) — so that error-prone single-molecule long reads (PacBio, Oxford Nanopore) do not carry strand-specific artifacts into downstream haplotype-informed genotyping.

## When to use

Use when candidate SNVs from error-prone long-read alignments (BAM/CRAM of PacBio SMRT or Oxford Nanopore reads) must be filtered for strand bias before haplotype assembly or genotyping.

## When NOT to use

- Input variants are already filtered final genotype calls with strand-bias annotations — re-filtering may remove true positives without benefit
- The data are short reads (Illumina) from tools like GATK HapCUT2Caller or FreeBayes, where the long-read strand-bias error model of Longshot does not apply
- Calling structural variants or indels de novo — Longshot only calls SNVs (indels can be genotyped only if supplied in an input VCF)

## Inputs

- Sorted, indexed BAM/CRAM file of error-prone long reads (PacBio SMRT or Oxford Nanopore)
- Indexed FASTA reference genome the reads were aligned to
- VCF of candidate/potential SNVs (gzipped, tabix-indexed when supplied via --potential_variants)

## Outputs

- Filtered candidate SNV VCF retaining only variants with Fisher's exact two-tailed p-value >= 0.01
- Final phased VCF of called variants (after haplotype-informed genotyping)

## How to apply

For each candidate SNV in the input VCF, count ref and alt allele observations separately on forward- and reverse-strand reads from the sorted, indexed BAM/CRAM, forming a 2x2 contingency table of allele by strand. Compute a two-tailed Fisher's exact test p-value for the allele-strand association and drop candidates with p < 0.01 (Longshot's --strand_bias_pvalue_cutoff default), since context-specific Nanopore and PacBio errors can appear strand-biased. Apply this filter before haplotype-informed genotyping so biased candidates never enter the HapCUT2-based joint genotyping. Retained candidates preserve chromosomal coordinates and alleles and are written to a filtered candidate SNV VCF. The filter complements other Longshot cutoffs — min_allele_qual (default 7.0), min_mapq (default 20), and min/max coverage (defaults 6 and 8000) — that reduce effective read depth and remove low-quality allele observations.

## Related tools

- **Longshot** (Implements the strand-bias Fisher's exact filter (--strand_bias_pvalue_cutoff, two-tailed p < 0.01) on candidate SNVs prior to haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Every variant in the filtered candidate VCF has a two-tailed Fisher's exact test p-value >= 0.01
- No retained candidate shows allele observations concentrated on a single read strand (2x2 allele-by-strand table is balanced)
- Filtered candidate VCF preserves chromosomal coordinates and REF/ALT alleles from the input candidate list
- Strand-biased candidates are excluded before HapCUT2 haplotype-informed genotyping, not after
- Called-variant accuracy against a truth set (e.g., ground_truth_variants.vcf in example_data) is not degraded compared to unfiltered genotyping

## Limitations

- Allele-quality filtering (phred-scaled quality score below 7.0 by default) discards allele observations, reducing the effective read depth for SMS reads
- Longshot currently calls only SNVs; indels are genotyped but not phased, and structural variants (length > 50 bp) are unsupported
- Excessive false potential variants in the input VCF can lead to inaccurate results when genotyping supplied variants

## Evidence

- [discussion] Fisher's exact test two-tailed p-value cutoff: [evidence span withheld — anchor status: not_found]
- [full_text] strand-bias filter rationale: [evidence span withheld — anchor status: not_found]
- [readme] input BAM requirements: "sorted, indexed BAM file with error-prone reads (CRAM files also supported)"
- [readme] companion mapping-quality filter: "Minimum mapping quality to use a read. [default: 20]"
