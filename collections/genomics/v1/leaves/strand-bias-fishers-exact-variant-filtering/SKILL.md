---
name: strand-bias-fishers-exact-variant-filtering
description: Use when calling SNVs from single-molecule long reads (PacBio, Oxford
  Nanopore) with Longshot and you need to remove false variant sites where observed
  alleles are over-represented on one sequencing strand.
when_to_use_negative:
- Calling short indels with SMS reads — the paper notes accurate short indel calling
  is challenging due to the high insertion/deletion error rate, and Longshot only
  calls SNVs de novo
- Processing short-read data (Illumina) where tools like GATK HaplotypeCaller or FreeBayes
  are the appropriate callers
- Genotyping a pre-specified VCF with -v where the paper warns that excessive false
  potential variants can lead to inaccurate results and every input variant is used
  regardless of filters
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Variant caller that internally applies the two-tailed Fisher's exact strand-bias
    test (p < 0.01 default) to candidate SNVs before haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:c15a93d5761d7d4fea403f198f63e83adc0099e716aafd52645ca93b90a75e3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Strand-bias Fisher's exact variant filtering

## Summary

A Longshot default filter that discards candidate SNVs whose allele observations are strand-biased, using a two-tailed Fisher's exact test at p < 0.01, which is especially important for error-prone Oxford Nanopore reads.

## When to use

Use when calling SNVs from single-molecule long reads (PacBio, Oxford Nanopore) with Longshot and you need to remove false variant sites where observed alleles are over-represented on one sequencing strand.

## When NOT to use

- Calling short indels with SMS reads — the paper notes accurate short indel calling is challenging due to the high insertion/deletion error rate, and Longshot only calls SNVs de novo
- Processing short-read data (Illumina) where tools like GATK HaplotypeCaller or FreeBayes are the appropriate callers
- Genotyping a pre-specified VCF with -v where the paper warns that excessive false potential variants can lead to inaccurate results and every input variant is used regardless of filters

## Inputs

- minimap2- or BLASR-aligned BAM/CRAM of long SMS reads (PacBio or Oxford Nanopore) against an indexed reference FASTA (e.g., GRCh38/hg38 or hs37d5)

## Outputs

- Filtered candidate SNV set entering haplotype-informed genotyping (strand-biased sites removed before joint phased genotyping)
- Final phased VCF of SNV calls free of strand-biased false positives

## How to apply

During pileup-based candidate SNV identification, Longshot tallies alt-allele-supporting reads by strand and applies a two-tailed Fisher's exact test; candidate SNVs with p-value < 0.01 are not considered for haplotype-informed genotyping. This filter is active by default (--strand_bias_pvalue_cutoff 0.01) and the paper notes it is the recommended setting for ONT reads because strand-biased allele observations are a signature of false SNV sites. Keep it enabled alongside the other default read filters (min_mapq 20, min_allele_qual 7.0, min_cov 6) and the depth-based max-coverage cutoff (d + 5*sqrt(d) median coverage) that excludes copy-number-amplified regions. No manual tuning is required for standard whole-genome ONT runs; override the cutoff only if you can justify it with calibration against a truth set such as GIAB high-confidence calls.

## Related tools

- **Longshot** (Variant caller that internally applies the two-tailed Fisher's exact strand-bias test (p < 0.01 default) to candidate SNVs before haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam ont.bam --ref grch38.fa --out na12878_ont_longshot.vcf
```

## Evaluation signals

- Fisher's exact test p-value for each candidate SNV's strand contingency table is ≥ 0.01 (two-tailed) for all sites retained in the output VCF
- Against the GIAB GRCh38 high-confidence call set for NA12878 ONT data at GQ 65, precision reaches ~0.991 and recall ~0.933 (best F1 0.967)
- Filter remains active when running Longshot with default options (strand_bias_pvalue_cutoff 0.01); verify by inspecting --help output rather than assuming
- No excess of variants in copy-number-amplified regions, since the strand-bias filter is complemented by the d + 5*sqrt(d) max read depth cutoff

## Limitations

- Allele quality filtering (threshold 7.0 by default) and strand-bias filtering reduce the effective read depth for SMS reads
- Some Longshot false positives surviving filters may actually correspond to false negatives in the GIAB high-confidence call sets, which are biased in favor of short-read technologies
- Correctly mapping long reads in highly similar segmental duplications remains a challenge and can confound downstream filtering/accuracy assessment

## Evidence

- [discussion] Fisher's exact strand-bias cutoff at default: [evidence span withheld — anchor status: not_found]
- [other] recommended for ONT reads: [evidence span withheld — anchor status: not_found]
- [readme] default cutoff in CLI: "Longshot: variant caller (SNVs) for long-read sequencing data"
- [discussion] candidate SNV identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [discussion] effective depth reduction from allele filtering: [evidence span withheld — anchor status: not_found]
