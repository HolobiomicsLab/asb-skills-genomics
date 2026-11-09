---
name: giab-benchmark-region-intersection
description: Use when comparing variant caller performance across genomic strata,
  e.g., quantifying how many NA12878 SNVs fall in segmental duplications vs GIAB high-confidence
  regions and computing Ts/Tv ratios per stratum.
when_to_use_negative:
- The goal is calling indels or structural variants — Longshot only calls SNVs and
  accurate short indel calling with SMS reads is challenging due to the high insertion/deletion
  error rate
- The sample lacks GIAB high-confidence benchmarks or segmental-duplication annotations,
  so strata cannot be defined
- Reads are short Illumina reads alone and the question concerns duplicated regions
  — Illumina calls are unreliable there (Ts/Tv 1.55 vs 1.99 for SMS)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Longshot
  role: Calls phased SNVs from the 30x PacBio SMS BAM with default parameters (min_cov
    6, max_cov 8000, min_mapq 20, min_alt_count 3, min_alt_frac 0.125)
  repo: https://github.com/pjedge/longshot
- name: FreeBayes
  role: Calls SNVs from the 33x Illumina BWA-MEM BAM as the short-read comparison
    call set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:abc3dd5b3f5faa2eb3246b48be0c18d766387784f4f2c58c77b2293e717054da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - FreeBayes
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# GIAB benchmark region intersection

## Summary

Stratify variant call sets (e.g., Longshot PacBio SNVs vs FreeBayes Illumina SNVs on NA12878) by intersecting them with GIAB high-confidence regions, their complements, and segmental-duplication annotation tracks at identity thresholds of ≥95% and ≥99%, then compute per-stratum SNV counts and Ts/Tv ratios.

## When to use

Use when comparing variant caller performance across genomic strata, e.g., quantifying how many NA12878 SNVs fall in segmental duplications vs GIAB high-confidence regions and computing Ts/Tv ratios per stratum.

## When NOT to use

- The goal is calling indels or structural variants — Longshot only calls SNVs and accurate short indel calling with SMS reads is challenging due to the high insertion/deletion error rate
- The sample lacks GIAB high-confidence benchmarks or segmental-duplication annotations, so strata cannot be defined
- Reads are short Illumina reads alone and the question concerns duplicated regions — Illumina calls are unreliable there (Ts/Tv 1.55 vs 1.99 for SMS)

## Inputs

- Longshot VCF of SNVs from 30x PacBio SMS reads aligned to hs37d5
- FreeBayes VCF of SNVs from 33x Illumina reads aligned to hs37d5 with BWA-MEM
- GIAB high-confidence region BED tracks for NA12878 (and their complements)
- Segmental duplication annotation tracks restricted to ≥95% and ≥99% identity thresholds

## Outputs

- Per-stratum SNV counts for each caller at each identity threshold (e.g., ≥95% and ≥99% duplications)
- Per-stratum Ts/Tv ratios for Longshot and FreeBayes calls, including the Illumina-only Ts/Tv in segmental duplications
- A summary table analogous to Table 2 of the Longshot paper

## How to apply

Call SNVs with each caller first: Longshot on the 30x PacBio SMS BAM (hs37d5, default min_cov 6, max_cov 8000, min_mapq 20) and FreeBayes on the 33x Illumina BWA-MEM BAM. Apply genotype-quality filtering — GQ cutoff fixed at 50 for short reads, a variable GQ cutoff matched to the median read coverage for Longshot — and remove false-positive clusters using the density rule (>10 SNVs in a 500 bp window) and a max read-depth filter of d+5*sqrt(d). Intersect both filtered VCFs with GIAB high-confidence regions, their complements, and segmental-duplication tracks restricted to ≥95% and ≥99% identity thresholds. Compute per-stratum SNV counts and transition/transversion (Ts/Tv) ratios per caller; compare call counts and the Illumina-only Ts/Tv in duplications to assess mappability bias.

## Related tools

- **Longshot** (Calls phased SNVs from the 30x PacBio SMS BAM with default parameters (min_cov 6, max_cov 8000, min_mapq 20, min_alt_count 3, min_alt_frac 0.125)) — https://github.com/pjedge/longshot
- **FreeBayes** (Calls SNVs from the 33x Illumina BWA-MEM BAM as the short-read comparison call set)

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- SNV counts reproduce Table 2 pattern: 180,889 (SMS) vs 116,649 (Illumina) at ≥95% identity, and 78,851 vs 18,684 (4.2-fold) at ≥99% identity
- At ≥95% similarity, SMS calls outnumber Illumina calls by ~55.0%
- Illumina-only Ts/Tv in ≥99% duplications is 1.55 versus 1.99 for SMS-only calls
- Filtered call sets contain no clusters violating the density rule (>10 SNVs in a 500 bp window) or depth > d+5*sqrt(d)
- Both call sets intersect cleanly with GIAB high-confidence regions and their complements without coordinate/reference mismatch

## Limitations

- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- GIAB and Platinum Genomes variant sets were generated using short-read datasets and are biased in favor of short-read technologies, so some Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets
- Allele observations with phred-scaled quality below 7.0 are discarded, reducing the effective read depth for SMS reads

## Evidence

- [discussion] GIAB high-confidence call sets are biased toward short reads: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets, are therefore biased in favor of short-read technologies"
- [discussion] GQ cutoff of 50 for short-read variants: "The GQ cutoff was fixed at 50 for short reads."
- [discussion] Density filter rule: [evidence span withheld — anchor status: not_found]
- [discussion] Max read depth filter: [evidence span withheld — anchor status: not_found]
- [abstract] Longshot enables variant detection in duplicated regions: "enables variant detection in duplicated regions of the genome that cannot be mapped using short reads"
- [readme] Longshot invocation from README: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
