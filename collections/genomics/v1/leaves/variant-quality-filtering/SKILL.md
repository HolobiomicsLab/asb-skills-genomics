---
name: variant-quality-filtering
description: Use when comparing or validating SNV call sets from Longshot (PacBio
  SMS reads) and FreeBayes (Illumina reads), or when filtering candidate SNVs by genotype
  quality, read depth, density, or strand bias before counting or intersecting with
  GIAB/segmental-duplication regions.
when_to_use_negative:
- Input is raw sequencing reads rather than an existing VCF of candidate SNVs — variant
  calling (Longshot/FreeBayes) must happen first
- The goal is structural variant or copy-number calling — Longshot only calls SNVs,
  and these filters target SNV false positives
- Comparing call sets without aligning caller-specific GQ cutoffs — the fixed GQ=50
  short-read cutoff vs. the variable Longshot cutoff must both be applied or counts
  are not comparable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Calls and genotypes SNVs from PacBio SMS reads; exposes quality-related parameters
    (min_cov 6, max_cov 8000, min_mapq 20, min_allele_qual, --auto_max_cov) that feed
    the filtering scheme
  repo: https://github.com/pjedge/longshot
- name: FreeBayes
  role: Short-read variant caller whose output is filtered with the fixed GQ cutoff
    of 50 for comparison with Longshot calls
- name: Delly
  role: Its exclusion regions (delly exclusion templates) are filtered out for the
    AJ trio Mendelian consistency analysis
  repo: https://github.com/tobiasrausch/delly/blob/master/excludeTemplates/human.hg38.excl.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:865cd604b18b69f0f7b740e26360f373a30bca21a5506eb38999124b93469f71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Delly
  - FreeBayes
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variant-quality-filtering

## Summary

Apply paper-specific quality filters to SNV call sets so that short-read (FreeBayes) and long-read (Longshot) calls are comparable: genotype-quality cutoffs, read-depth ceilings, SNV density and strand-bias filters, and optional common-indel filtering. This reduces coverage-driven and mapping-driven false positives, especially in segmental duplications.

## When to use

Use when comparing or validating SNV call sets from Longshot (PacBio SMS reads) and FreeBayes (Illumina reads), or when filtering candidate SNVs by genotype quality, read depth, density, or strand bias before counting or intersecting with GIAB/segmental-duplication regions.

## When NOT to use

- Input is raw sequencing reads rather than an existing VCF of candidate SNVs — variant calling (Longshot/FreeBayes) must happen first
- The goal is structural variant or copy-number calling — Longshot only calls SNVs, and these filters target SNV false positives
- Comparing call sets without aligning caller-specific GQ cutoffs — the fixed GQ=50 short-read cutoff vs. the variable Longshot cutoff must both be applied or counts are not comparable

## Inputs

- VCF of SNVs called by Longshot from 30x PacBio SMS reads aligned to hs37d5
- VCF of SNVs called by FreeBayes from 33x Illumina reads aligned with BWA-MEM
- Sorted, indexed BAM/CRAM files with long error-prone reads (for computing median read depth d and applying Longshot filters)
- Known common indel database (optional, for FDR reduction)

## Outputs

- Filtered, quality-controlled VCF of SNVs per caller (short-read GQ>=50; Longshot variable GQ matched to median coverage)
- Per-stratum SNV counts and Ts/Tv ratios (e.g., in segmental duplications at >=95% and >=99% identity) after filtering
- Exclusion lists of density-flagged clusters (>10 SNVs per 500 bp) and high-depth variants (>d+5*sqrt(d))

## How to apply

Filter short-read variants with a fixed GQ cutoff of 50, while for Longshot use a variable GQ cutoff matched to the median read coverage to keep false-discovery rates comparable between technologies. Remove variants with read depth above d + 5*sqrt(d), where d is the median read depth across the dataset, to eliminate copy-number-amplification false SNVs. Drop low-density false-positive clusters flagged by the density rule (>10 SNVs in a 500 bp window). For candidate sites entering haplotype-informed genotyping, discard SNVs with Fisher's exact test (two-tailed) p < 0.01 for strand bias and allele observations with phred-scaled quality below 7.0. Optionally filter known common indels, which reduces the Longshot false-discovery rate to 0.3%. Note that Longshot's own default calling parameters include min_cov 6, max_cov 8000, min_mapq 20, and its --auto_max_cov flag sets max coverage to mean_coverage + 5*sqrt(mean_coverage).

## Related tools

- **Longshot** (Calls and genotypes SNVs from PacBio SMS reads; exposes quality-related parameters (min_cov 6, max_cov 8000, min_mapq 20, min_allele_qual, --auto_max_cov) that feed the filtering scheme) — https://github.com/pjedge/longshot
- **FreeBayes** (Short-read variant caller whose output is filtered with the fixed GQ cutoff of 50 for comparison with Longshot calls)
- **Delly** (Its exclusion regions (delly exclusion templates) are filtered out for the AJ trio Mendelian consistency analysis) — https://github.com/tobiasrausch/delly/blob/master/excludeTemplates/human.hg38.excl.tsv

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- Reproduce the Table 2 stratified tallies: 180,889 vs 116,649 SNVs at >=95% segmental-duplication identity and 78,851 vs 18,684 at >=99% identity
- Illumina-only Ts/Tv ratio in >=99% identity duplications equals 1.55 versus 1.99 for SMS-only calls after filtering
- No retained variant exceeds read depth d + 5*sqrt(d), where d is the median read depth of the dataset
- No retained call window contains more than 10 SNVs within 500 bp
- Filtering known common indels drives the Longshot FDR down to ~0.3%

## Limitations

- Allele observations with phred-scaled quality below 7.0 are discarded, which reduces the effective read depth for SMS reads
- Some Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets, since those call sets are biased in favor of short-read technologies
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, so residual false positives in duplicated regions are possible even after filtering

## Evidence

- [discussion] Fixed vs. variable GQ cutoffs for short and long reads: "The GQ cutoff was fixed at 50 for short reads."
- [discussion] Variable GQ for Longshot matched to median coverage: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] Read-depth ceiling removes copy-number false SNVs: [evidence span withheld — anchor status: not_found]
- [discussion] Density filter for clustered false variants: [evidence span withheld — anchor status: not_found]
- [discussion] Strand-bias filter via Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [discussion] Common-indel filtering reduces FDR: [evidence span withheld — anchor status: not_found]
- [methods] Longshot default calling parameters: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
