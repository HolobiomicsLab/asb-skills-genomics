---
name: fishers-exact-test-variant-quality-control
description: Use when filtering candidate SNV lists derived from single-molecule long
  reads (PacBio, Oxford Nanopore) whose allele observations are significantly over-represented
  on one read strand prior to haplotype assembly or genotyping.
when_to_use_negative:
- Input variants are already final genotype calls rather than candidate SNVs pending
  haplotype-informed genotyping — this filter is applied pre-genotyping in Longshot's
  pipeline
- The goal is structural variant discovery (e.g., Delly) rather than SNV candidate
  quality control
- 'Indel-only variant sets: Longshot only calls SNVs and the strand-bias filter targets
  SNV candidates'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Implements the two-tailed Fisher's exact test strand-bias filter (--strand_bias_pvalue_cutoff,
    default 0.01) on candidate SNVs before haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:ed7a2b3765a201f900592b2384b5984674ec9d8d6d0bc341217be06a365f156f
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
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# fishers-exact-test-variant-quality-control

## Summary

A strand-bias filter for candidate SNVs in error-prone long-read data: build a 2x2 ref/alt-by-forward/reverse-strand contingency table at each candidate site and drop variants whose two-tailed Fisher's exact test p-value is < 0.01 before haplotype-informed genotyping, countering context-specific Oxford Nanopore errors.

## When to use

Use when filtering candidate SNV lists derived from single-molecule long reads (PacBio, Oxford Nanopore) whose allele observations are significantly over-represented on one read strand prior to haplotype assembly or genotyping.

## When NOT to use

- Input variants are already final genotype calls rather than candidate SNVs pending haplotype-informed genotyping — this filter is applied pre-genotyping in Longshot's pipeline
- The goal is structural variant discovery (e.g., Delly) rather than SNV candidate quality control
- Indel-only variant sets: Longshot only calls SNVs and the strand-bias filter targets SNV candidates

## Inputs

- Sorted, indexed BAM/CRAM of error-prone long reads (PacBio SMRT or Oxford Nanopore)
- Allelotyped candidate SNV list (gzipped, tabix-indexed VCF) with chromosomal coordinates and ref/alt alleles

## Outputs

- filtered_candidate_snvs.vcf containing only candidate SNVs with two-tailed Fisher's exact test p-value >= 0.01, preserving coordinates and alleles

## How to apply

For each candidate SNV in the allelotyped VCF, count reference and alternate allele observations separately in forward-strand and reverse-strand reads from the sorted, indexed BAM/CRAM. Construct a 2x2 contingency table (ref/alt alleles crossed with forward/reverse strand) and compute the two-tailed Fisher's exact test p-value for allele-strand association, as implemented by Longshot's --strand_bias_pvalue_cutoff option. Remove any candidate whose p-value falls below the default cutoff of 0.01 so strand-biased candidates are excluded from HapCUT2-based haplotype-informed genotyping. The rationale is that Nanopore errors are context-specific and can create spurious allele observations concentrated on one strand.

## Related tools

- **Longshot** (Implements the two-tailed Fisher's exact test strand-bias filter (--strand_bias_pvalue_cutoff, default 0.01) on candidate SNVs before haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- Every variant retained in filtered_candidate_snvs.vcf has a two-tailed Fisher's exact test p-value >= 0.01
- Strand-biased candidate SNVs are excluded prior to haplotype-informed genotyping (absent from downstream phased VCF)
- Retained candidates preserve original chromosomal coordinates and ref/alt alleles
- After running the full Longshot pipeline, retained variants show balanced allele support across forward and reverse strands

## Limitations

- The filter discards candidates with p < 0.01, which may remove genuine variants whose errors are strand-concentrated; allele observations with phred-scaled quality below 7.0 are already discarded, reducing effective read depth for SMS reads
- Aimed at SNVs; short indel calling with SMS reads remains challenging due to high insertion/deletion error rates

## Evidence

- [discussion] Strand bias filter p<0.01: [evidence span withheld — anchor status: not_found]
- [discussion] Strand-bias filter workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Nanopore error rationale: "Single-molecule sequencing (SMS) technologies such as Pacific Biosciences and Oxford Nanopore generate long reads"
- [readme] Input VCF format requirement: "NOTES: VCF must be gzipped and tabix indexed or contain contig information. Use with caution because excessive false potential variants can lead to inaccurate results."
- [discussion] Candidate SNV identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
