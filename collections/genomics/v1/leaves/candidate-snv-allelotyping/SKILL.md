---
name: candidate-snv-allelotyping
description: Use when calling SNVs from error-prone single-molecule long reads (PacBio
  SMRT or Oxford Nanopore) and candidate variants from a pileup or input VCF must
  be allelotyped and filtered for strand bias prior to haplotype-informed genotyping.
when_to_use_negative:
- De novo short indel discovery from SMS reads — Longshot only calls SNVs and can
  genotype indels only if supplied in an input VCF; indel calling is challenged by
  the high insertion/deletion error rate
- Structural variant discovery — Longshot does not support structural variants (length
  > 50 bp)
- Candidate variants are already haplotype-informed genotyped and strand-bias screened,
  so re-allelotyping would discard legitimate candidates or waste computation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Performs candidate SNV allelotyping from the long-read pileup, applies the
    two-tailed Fisher's exact strand-bias filter (p < 0.01 default), and writes the
    filtered candidate SNVs for haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:533eee4a596708ad8472475337510c84a528a7690ab59e2841fe0e8aba7d62fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# candidate-snv-allelotyping

## Summary

Allelotypes candidate SNVs from aligned error-prone long reads (PacBio SMRT, Oxford Nanopore) by counting ref/alt allele observations per read, applying coverage, mapping-quality, allele-quality, and strand-bias filters (two-tailed Fisher's exact p >= 0.01) before haplotype-informed genotyping with Longshot.

## When to use

Use when calling SNVs from error-prone single-molecule long reads (PacBio SMRT or Oxford Nanopore) and candidate variants from a pileup or input VCF must be allelotyped and filtered for strand bias prior to haplotype-informed genotyping.

## When NOT to use

- De novo short indel discovery from SMS reads — Longshot only calls SNVs and can genotype indels only if supplied in an input VCF; indel calling is challenged by the high insertion/deletion error rate
- Structural variant discovery — Longshot does not support structural variants (length > 50 bp)
- Candidate variants are already haplotype-informed genotyped and strand-bias screened, so re-allelotyping would discard legitimate candidates or waste computation

## Inputs

- sorted, indexed BAM/CRAM of error-prone long reads (PacBio SMRT or Oxford Nanopore)
- indexed reference genome FASTA aligned to
- optional gzipped, tabix-indexed VCF of potential variants to genotype (--potential_variants)

## Outputs

- filtered candidate SNV VCF with chromosomal coordinates and alleles preserved (all retained variants have two-tailed Fisher's exact p-value >= 0.01)
- phased VCF of called variants with haplotype information
- optional haplotype-tagged BAM (HP:i:1 / HP:i:2) via --out_bam

## How to apply

Run Longshot on a sorted, indexed BAM/CRAM of long reads with an indexed reference FASTA; it first identifies potential SNVs from the pileup by a genotype likelihood calculation (similar to Samtools), subject to min_cov (default 6), max_cov (default 8000), min_mapq (default 20), and allele quality >= 7.0 filters. At each candidate site, count ref and alt allele observations separately on forward- and reverse-strand reads and build a 2x2 allele-by-strand contingency table. Compute the two-tailed Fisher's exact test p-value for allele-strand association and drop candidates with p < 0.01 (the default --strand_bias_pvalue_cutoff), because Nanopore context-specific errors can be strand-biased and would otherwise feed false candidates into haplotype-informed genotyping. Retained candidates, preserving chromosomal coordinates and alleles, are written to a filtered candidate VCF and subsequently genotyped jointly via HapCUT2-based iterative phasing and local genotype updates.

## Related tools

- **Longshot** (Performs candidate SNV allelotyping from the long-read pileup, applies the two-tailed Fisher's exact strand-bias filter (p < 0.01 default), and writes the filtered candidate SNVs for haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Every variant in filtered_candidate_snvs.vcf has a two-tailed Fisher's exact p-value >= 0.01; strand-biased candidates are excluded before haplotype-informed genotyping
- Coverage and quality filters were applied at their defaults: min_cov 6, max_cov 8000, min_mapq 20, min_allele_qual 7.0 (discarding allele observations below threshold)
- Filtered candidate VCF preserves chromosomal coordinates and reference/alternate alleles from the candidate list
- On the example_data toy dataset, calls match the 714 ground-truth variants in example_data/ground_truth_variants.vcf
- Read depth per candidate stays within the max-cov ceiling (or mean coverage + 5*sqrt(mean coverage) with --auto_max_cov) to exclude copy-number amplification false SNVs

## Limitations

- Discarding allele observations with phred-scaled quality below 7.0 reduces the effective read depth for SMS reads, which can lower sensitivity at low coverage
- Strand-bias filtering addresses context-specific Nanopore errors, but accurately calling short indels with SMS reads remains challenging due to the high insertion/deletion error rate
- Excessive false potential variants supplied via --potential_variants can lead to inaccurate genotyping results

## Evidence

- [discussion] Strand bias filter rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Candidate SNV identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [readme] Coverage filter default: "-c, --min_cov <int>                            Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
- [readme] Example execution command: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
