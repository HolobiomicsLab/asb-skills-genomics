---
name: strand-bias-statistical-filtering
description: Use when calling SNVs from error-prone Oxford Nanopore or PacBio SMRT
  reads and candidate variants show alleles over-represented on one read strand; the
  filter runs on the allelotyped candidate list prior to haplotype assembly.
when_to_use_negative:
- Input is short-read data — the article positions this filter for SMS reads; short-read
  variant calling uses tools like GATK HaplotyperCaller and FreeBayes
- Calling variant classes other than SNVs — Longshot currently only calls single nucleotide
  variants (indels can only be genotyped if supplied in an input VCF)
- All candidate variants must be retained regardless of strand skew (e.g. with --potential_variants,
  since 'Every variant is used and only the allele fields are considered') — filtering
  would discard intended input sites
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Implements the strand-bias filter (two-tailed Fisher's exact test, --strand_bias_pvalue_cutoff,
    default 0.01) on candidate SNVs before haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:cf32a102b854f2d7ded3eedb4395c4e246472549ed12c618243015b253ed5a64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# strand-bias-statistical-filtering

## Summary

A candidate-SNV filtering step from Longshot that removes potential variants whose allele observations show significant strand bias, judged by a two-tailed Fisher's exact test p-value < 0.01. It counters context-specific, strand-dependent sequencing errors in error-prone single-molecule sequencing (SMS) reads before haplotype-informed genotyping.

## When to use

Use when calling SNVs from error-prone Oxford Nanopore or PacBio SMRT reads and candidate variants show alleles over-represented on one read strand; the filter runs on the allelotyped candidate list prior to haplotype assembly.

## When NOT to use

- Input is short-read data — the article positions this filter for SMS reads; short-read variant calling uses tools like GATK HaplotyperCaller and FreeBayes
- Calling variant classes other than SNVs — Longshot currently only calls single nucleotide variants (indels can only be genotyped if supplied in an input VCF)
- All candidate variants must be retained regardless of strand skew (e.g. with --potential_variants, since 'Every variant is used and only the allele fields are considered') — filtering would discard intended input sites

## Inputs

- sorted, indexed BAM/CRAM file of error-prone long reads (Oxford Nanopore or PacBio SMRT) aligned to an indexed FASTA reference
- allelotyped candidate SNV list (gzipped, tabix-indexed VCF, e.g. via the --potential_variants option or Longshot's internal pileup-based candidate discovery)

## Outputs

- filtered candidate SNV VCF retaining only candidates with two-tailed Fisher's exact p-value >= 0.01, preserving chromosomal coordinates and alleles
- downstream phased VCF of haplotype-informed genotypes that excludes strand-biased candidate variants

## How to apply

Load the sorted, indexed BAM/CRAM of long SMS reads and the allelotyped candidate SNV list, and for each candidate site count ref and alt allele observations separately on forward- and reverse-strand reads. Build the 2x2 contingency table of ref/alt alleles crossed with read strand and compute the two-tailed Fisher's exact test p-value for allele-strand association, as implemented by Longshot's --strand_bias_pvalue_cutoff option (default 0.01). Drop any candidate whose p-value falls below the cutoff so strand-biased candidates are never considered for haplotype-informed genotyping; Longshot is configured to use only reads passing filters (min_mapq default 20, min_allele_qual default 7.0) when building these observations. Write the retained candidates, preserving chromosomal coordinates and alleles, to the filtered candidate SNV VCF, then proceed to pair-HMM local realignment and HapCUT2-based iterative phasing/genotyping. The rationale is that genuine heterozygous SNVs should have alt alleles observed on both strands, whereas context-specific Nanopore or PacBio errors tend to be strand-concentrated.

## Related tools

- **Longshot** (Implements the strand-bias filter (two-tailed Fisher's exact test, --strand_bias_pvalue_cutoff, default 0.01) on candidate SNVs before haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Every variant in the filtered candidate SNV VCF has a two-tailed Fisher's exact test p-value >= 0.01 for allele-strand association
- Strand-biased candidates are absent from the haplotype-informed genotyping step and from the final phased VCF
- Chromosomal coordinates and ref/alt alleles in the filtered candidate VCF match the original allelotyped candidate list for all retained sites
- The filtered set is a strict subset of the unfiltered candidate set, with only low-p-value (strand-biased) sites removed
- Allele observations used for the contingency tables come only from reads passing the configured filters (min_mapq 20, min_allele_qual 7.0, min_cov 6 by default)

## Limitations

- Discarding strand-biased candidates plus low-quality allele observations (phred-scaled quality below 7.0 by default) reduces the effective read depth for SMS reads
- The 0.01 p-value cutoff is a fixed default; genuine variants with strand-concentrated coverage (e.g. local alignment artifacts) would also be removed
- Longshot's false positives relative to GIAB high-confidence call sets may partly reflect short-read bias in the benchmark sets, so filter performance against truth sets must be interpreted cautiously

## Evidence

- [discussion] Strand bias filter drops p<0.01 candidates before genotyping: [evidence span withheld — anchor status: not_found]
- [readme] Longshot targets long error-prone SMS reads: "Longshot is a variant calling tool for diploid genomes using long error prone reads such as Pacific Biosciences (PacBio) SMRT and Oxford Nanopore Technologies (ONT)."
- [discussion] Allele quality threshold reduces effective depth: [evidence span withheld — anchor status: not_found]
- [readme] Input BAM must be sorted and indexed: "-b, --bam <BAM>                            sorted, indexed BAM file with error-prone reads (CRAM files also supported)"
