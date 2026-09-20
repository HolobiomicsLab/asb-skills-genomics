---
name: long-read-pileup-allele-counting
description: Use when calling SNVs from sorted, indexed BAM/CRAM files of error-prone
  single-molecule long reads and you need to vet candidate variants (e.g. from allelotyping
  or an input VCF) for context-specific Nanopore strand-bias artifacts prior to haplotype
  assembly.
when_to_use_negative:
- Variant calls have already been genotyped and filtered by Longshot — the strand-bias
  screen targets candidates prior to haplotype-informed genotyping, not final calls.
- Calling short indels from SMS reads, which the article notes is challenging due
  to the high insertion/deletion error rate and outside Longshot's SNV-only calling
  scope.
- Input reads are short reads (Illumina) where strand bias artifacts of Oxford Nanopore
  error context do not apply.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: counts allele observations from the read pileup, applies the strand-bias Fisher's
    exact test (--strand_bias_pvalue_cutoff, default 0.01), and writes the filtered
    candidate SNV list for haplotype-informed genotyping
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:16b972b33b9894582501105e8d64c40a9d4755d58d871c06105d8220e739a41c
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# long-read-pileup-allele-counting

## Summary

Count reference and alternate allele observations per candidate SNV from the pileup of aligned error-prone long reads (PacBio SMRT / Oxford Nanopore), stratified by forward/reverse strand, and remove strand-biased candidates via a two-tailed Fisher's exact test (p < 0.01) before haplotype-informed genotyping with Longshot.

## When to use

Use when calling SNVs from sorted, indexed BAM/CRAM files of error-prone single-molecule long reads and you need to vet candidate variants (e.g. from allelotyping or an input VCF) for context-specific Nanopore strand-bias artifacts prior to haplotype assembly.

## When NOT to use

- Variant calls have already been genotyped and filtered by Longshot — the strand-bias screen targets candidates prior to haplotype-informed genotyping, not final calls.
- Calling short indels from SMS reads, which the article notes is challenging due to the high insertion/deletion error rate and outside Longshot's SNV-only calling scope.
- Input reads are short reads (Illumina) where strand bias artifacts of Oxford Nanopore error context do not apply.

## Inputs

- sorted, indexed BAM or CRAM of error-prone long reads (PacBio SMRT or Oxford Nanopore)
- indexed FASTA reference genome the reads are aligned to
- gzipped, tabix-indexed VCF of allelotyped candidate SNVs (or internal pileup-derived candidates)

## Outputs

- filtered_candidate_snvs.vcf with retained candidate SNVs preserving chromosomal coordinates and alleles
- per-site 2x2 allele-by-strand contingency counts and Fisher's exact p-values

## How to apply

At each candidate SNV site, tally reference and alternate allele observations separately for forward- and reverse-strand reads, applying Longshot's read-level filters (min_mapq 20 by default, min_allele_qual 7.0, coverage bounded by min_cov 6 and max_cov 8000). Build a 2x2 contingency table of ref/alt alleles crossed with forward/reverse strand and compute the two-tailed Fisher's exact test p-value, as implemented by Longshot's --strand_bias_pvalue_cutoff option. Drop any candidate whose p-value falls below the default 0.01 cutoff so strand-biased artifacts are not passed to haplotype-informed genotyping; retained candidates are written preserving chromosomal coordinates and alleles. This step complements Longshot's other candidate filters (max read depth d+5√d and the >10 SNVs per 500 bp density filter) that guard against copy-number and clustered false variants.

## Related tools

- **Longshot** (counts allele observations from the read pileup, applies the strand-bias Fisher's exact test (--strand_bias_pvalue_cutoff, default 0.01), and writes the filtered candidate SNV list for haplotype-informed genotyping) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- Every variant in filtered_candidate_snvs.vcf must have a two-tailed Fisher's exact p-value >= 0.01 (default --strand_bias_pvalue_cutoff).
- Allele counts must come only from reads passing Longshot's filters: min_mapq 20, min_allele_qual 7.0, and coverage within min_cov 6 to max_cov 8000.
- Chromosomal coordinates and ref/alt alleles in the output VCF must match the candidate input VCF for retained sites.
- Validation against ground truth (e.g. the README's example_data/ground_truth_variants.vcf) should show strand-biased false candidates excluded before genotyping.

## Limitations

- Allele quality filtering (phred-scaled quality score below 7.0 by default is discarded) reduces the effective read depth for SMS reads.
- The max read depth filter d+5√d removes apparent copy-number amplification false SNVs, so genuine high-coverage candidates may be lost.
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, affecting pileup allele counts in those regions.

## Evidence

- [discussion] Strand bias filter rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Candidate identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [discussion] Read-level quality filter: [evidence span withheld — anchor status: not_found]
- [readme] Coverage cutoffs from README: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
- [discussion] Depth filter for CN artifacts: [evidence span withheld — anchor status: not_found]
