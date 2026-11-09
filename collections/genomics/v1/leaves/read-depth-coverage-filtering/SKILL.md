---
name: read-depth-coverage-filtering
description: Use when calling SNVs from error-prone long-read (PacBio CLR/ONT) BAM
  alignments where locally elevated read depth from segmental duplications or copy-number
  amplification would otherwise produce false-positive variant calls.
when_to_use_negative:
- When genotyping a pre-specified VCF of candidate variants with -v, where the goal
  is joint genotyping rather than de novo candidate discovery and coverage gating
  may drop true amplified-region alleles
- When the dataset is uniformly very low coverage (below the min_cov 6 default), since
  the filter will eliminate nearly all candidate positions rather than correcting
  for amplification
- For structural-variant discovery (e.g., Delly CNV calling), where read depth is
  the signal of interest rather than a nuisance to filter out
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Longshot
  role: SNV caller whose candidate-discovery stage applies the min_cov/max_cov coverage
    gates and the -A auto max-coverage cutoff (mean + 5*sqrt(mean))
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Phasing engine used inside Longshot after coverage-filtered candidate SNVs
    are retained; iterated with local genotype updates
- name: Delly
  role: Source of the human exclusion-region templates; exclusion regions are filtered
    for the Mendelian consistency analysis, complementing depth-based SNV filtering
  repo: https://github.com/dellytools/delly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:e365994062ebfad6a0e2f4d4832b152bd56e453fbb057fdb4a6dd49f7c83e895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Delly
  - HapCUT2
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# read-depth-coverage-filtering

## Summary

A coverage-based candidate-SNV filtering strategy from Longshot: positions are considered only if read depth falls between a minimum coverage and a maximum set as d + 5*sqrt(d) (median depth d), which suppresses false SNVs arising from copy-number amplifications in PacBio/ONT long-read alignments.

## When to use

Use when calling SNVs from error-prone long-read (PacBio CLR/ONT) BAM alignments where locally elevated read depth from segmental duplications or copy-number amplification would otherwise produce false-positive variant calls.

## When NOT to use

- When genotyping a pre-specified VCF of candidate variants with -v, where the goal is joint genotyping rather than de novo candidate discovery and coverage gating may drop true amplified-region alleles
- When the dataset is uniformly very low coverage (below the min_cov 6 default), since the filter will eliminate nearly all candidate positions rather than correcting for amplification
- For structural-variant discovery (e.g., Delly CNV calling), where read depth is the signal of interest rather than a nuisance to filter out

## Inputs

- Sorted, indexed BAM/CRAM file of long PacBio SMRT or Oxford Nanopore reads aligned to an indexed FASTA reference
- Indexed reference genome FASTA (e.g., hs37d5 or hg38)
- Median or mean read depth statistic for the dataset (computed internally or via --auto_max_cov)

## Outputs

- Filtered phased VCF of SNVs in which candidate sites above the coverage ceiling have been excluded
- Optional haplotype-tagged BAM (HP:i:1/HP:i:2) from Longshot for downstream phasing checks

## How to apply

Compute the median read depth d across the entire dataset (or let Longshot compute mean coverage per region with the -A/--auto_max_cov flag, which sets max coverage to mean_coverage + 5*sqrt(mean_coverage)), then discard candidate SNVs whose filtered-read depth exceeds d + 5*sqrt(d). The rationale is that copy-number-amplified loci inflate allele observations and drive spurious heterozygous calls. Coverage gating operates alongside default candidate filters in Longshot: min_cov 6, min_mapq 20, and an allele quality threshold of 7.0, and should be paired with the density filter (>10 SNVs in a 500 bp window) and the two-tailed Fisher's exact strand-bias filter (p < 0.01) for high-coverage PacBio GIAB data. Note that --auto_max_cov is slower because it scans the region, but it adapts the cutoff to the actual coverage of each chromosome region when genomes are processed separately with --region.

## Related tools

- **Longshot** (SNV caller whose candidate-discovery stage applies the min_cov/max_cov coverage gates and the -A auto max-coverage cutoff (mean + 5*sqrt(mean))) — https://github.com/pjedge/longshot
- **HapCUT2** (Phasing engine used inside Longshot after coverage-filtered candidate SNVs are retained; iterated with local genotype updates)
- **Delly** (Source of the human exclusion-region templates; exclusion regions are filtered for the Mendelian consistency analysis, complementing depth-based SNV filtering) — https://github.com/dellytools/delly

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf -A --min_cov 6 --min_mapq 20
```

## Evaluation signals

- On GIAB PacBio benchmarks (NA12878 30x, NA24385 28x-64x), reproduced precision/recall should match reported values such as NA12878 30x P=0.9942/R=0.9592 and NA24385 recall rising from 0.9608 at 28x to 0.9798 at 64x
- Applying the d + 5*sqrt(d) depth ceiling should remove the excess false SNVs concentrated in copy-number-amplified regions without materially reducing recall in normal-coverage regions
- Residual high-SNV-density sites flagged 'dn' (>10 SNVs per 500 bp) should be absent or explicitly noted in the output VCF
- Strand-bias filtering (two-tailed Fisher's exact p < 0.01) should eliminate variants over-represented on one strand among retained candidates
- Running with -A should yield per-region max coverage equal to mean_coverage + 5*sqrt(mean_coverage), verifiable against samtools depth statistics

## Limitations

- Allele observations with quality below the 7.0 default threshold are discarded, reducing the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, so depth filtering cannot fully resolve duplicated-region artifacts
- Some residual Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets, which are biased in favor of short-read technologies
- The --auto_max_cov flag is slower because it must compute mean coverage for each region

## Evidence

- [discussion] Max read depth filter removes copy-number amplification false SNVs: [evidence span withheld — anchor status: not_found]
- [discussion] Density filter removes clustered false SNVs: [evidence span withheld — anchor status: not_found]
- [discussion] Strand-bias filter definition: [evidence span withheld — anchor status: not_found]
- [readme] auto_max_cov flag semantics: "Automatically calculate mean coverage for region and set max coverage to mean_coverage + 5*sqrt(mean_coverage). (SLOWER)"
- [readme] Default candidate filters: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
