---
name: long-read-allelotyping
description: Use when you have a sorted, indexed BAM/CRAM of long error-prone reads
  (PacBio SMRT or Oxford Nanopore) aligned to an indexed FASTA reference and need
  to determine, for each read at a candidate SNV, which allele it supports and with
  what quality.
when_to_use_negative:
- The goal is calling short indels de novo from SMS reads — accurate indel calling
  is challenging due to the high insertion/deletion error rate, and Longshot only
  genotypes indels supplied in an input VCF
- Input reads are short reads (e.g., Illumina) — this skill's local realignment and
  error model target long error-prone SMS reads
- Variants are structural variants longer than 50 bp — Longshot does not support them
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Performs pair-HMM forward-algorithm local realignment of reads to ref/alt
    windows and computes per-read allele assignments and allele quality scores
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:a2ac82ed3f24685100ad964bd843da5e23de2060fc4226b53b09e946b20ab0dc
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

# long-read-allelotyping

## Summary

Per-read allelotyping at candidate biallelic SNV sites using Longshot's pair-HMM local realignment: each long SMS read overlapping a candidate variant is realigned to ref and alt window sequences, assigned to the more likely allele, and given a phred-scaled allele quality. This is the read-level step that feeds Longshot's haplotype-informed genotyping.

## When to use

Use when you have a sorted, indexed BAM/CRAM of long error-prone reads (PacBio SMRT or Oxford Nanopore) aligned to an indexed FASTA reference and need to determine, for each read at a candidate SNV, which allele it supports and with what quality.

## When NOT to use

- The goal is calling short indels de novo from SMS reads — accurate indel calling is challenging due to the high insertion/deletion error rate, and Longshot only genotypes indels supplied in an input VCF
- Input reads are short reads (e.g., Illumina) — this skill's local realignment and error model target long error-prone SMS reads
- Variants are structural variants longer than 50 bp — Longshot does not support them

## Inputs

- Sorted, indexed BAM/CRAM of long reads (PacBio SMRT or Oxford Nanopore)
- Indexed FASTA reference genome that the BAM is aligned to
- Optional gzipped, tabix-indexed VCF of candidate/potential variants to genotype (-v)

## Outputs

- Per-(read, SNV) allele observations table (allelotype_observations.tsv) with read identifier, SNV position, called allele, and phred-scaled allele quality
- Optional per-step variant debug files written to --variant_debug_dir

## How to apply

Load the sorted, indexed BAM and indexed FASTA, retaining only reads passing the min_mapq filter (default 20) at each candidate biallelic SNV. For each read-SNV pair, build the realignment window from indel-free anchor sequences of length 6 (anchor_length) plus up to 50 padding bases (max_window) on either side of the variant, and realign the read to the reference and alternate window versions using the pair-HMM forward algorithm with a minimum alignment band width of 20 to obtain p_ref and p_alt; averaging over all possible local alignments is what makes allelotyping robust to the high single-molecule sequencing error rate. Discard a read-variant if its window contains a CIGAR indel (I/D/N) longer than 20 (max_cigar_indel), then assign the allele with the higher likelihood and convert the ref/alt posterior ratio to a phred-scaled allele quality. Drop allele observations with quality below the min_allele_qual default of 7.0 — this filtering reduces effective read depth for SMS reads but removes unreliable observations. Optionally write per-(read, SNV) observations to allelotype_observations.tsv and enable per-step variant dumps via --variant_debug_dir; the -S/--stable_alignment logspace pair-HMM option trades speed for numerical stability.

## Related tools

- **Longshot** (Performs pair-HMM forward-algorithm local realignment of reads to ref/alt windows and computes per-read allele assignments and allele quality scores) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Output table allelotype_observations.tsv contains read identifier, SNV position, called allele, and phred-scaled quality fields
- No retained allele observation has phred-scaled quality below the default threshold of 7.0
- Each read is assigned to exactly one of the two biallelic alleles, matching the allele with the higher pair-HMM likelihood (p_ref vs p_alt)
- Read-SNV pairs whose realignment windows contain CIGAR indels (I/D/N) longer than 20 are absent from the output
- On the README example dataset, the resulting VCF can be compared to example_data/ground_truth_variants.vcf (714 true variants) for accuracy

## Limitations

- Allele quality filtering (phred < 7.0 by default) reduces the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- Longshot currently calls only SNVs; indels are genotyped only if provided in an input VCF, and structural variants (length > 50 bp) are not supported
- Excessive false potential variants supplied via -v can lead to inaccurate results since every variant is used

## Evidence

- [results] pair-HMM forward algorithm averages over alignments: [evidence span withheld — anchor status: not_found]
- [discussion] local realignment via pair-HMM: "This local realignment is performed using a pair-HMM29."
- [readme] min_mapq default: "Minimum mapping quality to use a read. [default: 20]"
- [discussion] allele quality filter rationale: [evidence span withheld — anchor status: not_found]
- [discussion] allelotyping combined with genotyping: [evidence span withheld — anchor status: not_found]
