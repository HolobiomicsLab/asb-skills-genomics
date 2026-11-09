---
name: snv-allele-observation-filtering
description: Use when per-read allele calls at candidate biallelic SNV sites from
  long error-prone SMS reads (PacBio SMRT or ONT) must be filtered to phred-scaled
  allele quality >= 7.0 before downstream phased genotyping.
when_to_use_negative:
- Calling short indels de novo from SMS reads — Longshot only calls SNVs and can genotype
  indels only when provided in an input VCF, and accurate short indel calling is difficult
  due to the high insertion/deletion error rate
- Genotyping structural variants longer than 50 bp, which Longshot does not support
- Short-read (Illumina) variant calling, where pair-HMM local realignment allelotyping
  of error-prone SMS reads is not the appropriate model
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Performs pair-HMM local realignment allelotyping of reads at candidate SNV
    sites and applies the min_mapq, max_cigar_indel, and min_allele_qual (7.0) filters
    to produce per-(read, SNV) allele observations
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: 'Downstream consumer context: Longshot uses HapCUT2 to estimate the most likely
    pair of haplotypes from the filtered allele observations during iterative phased
    genotyping'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:da9c56a438fd4fbb8a0804061fcbd6a4f17f8c4b6c2d73bbb6c282cbc9a6fc3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - HapCUT2
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# snv-allele-observation-filtering

## Summary

Filter per-(read, SNV) allele observations produced by Longshot's pair-HMM allelotyping so that only high-confidence allele assignments survive: discard reads below min_mapq 20, read-variant windows with CIGAR indel ops longer than 20, and allele observations with phred-scaled quality below 7.0. This reduces the effective read depth but removes error-prone SMS observations before haplotype-informed genotyping.

## When to use

Use when per-read allele calls at candidate biallelic SNV sites from long error-prone SMS reads (PacBio SMRT or ONT) must be filtered to phred-scaled allele quality >= 7.0 before downstream phased genotyping.

## When NOT to use

- Calling short indels de novo from SMS reads — Longshot only calls SNVs and can genotype indels only when provided in an input VCF, and accurate short indel calling is difficult due to the high insertion/deletion error rate
- Genotyping structural variants longer than 50 bp, which Longshot does not support
- Short-read (Illumina) variant calling, where pair-HMM local realignment allelotyping of error-prone SMS reads is not the appropriate model

## Inputs

- Sorted, indexed BAM or CRAM file of long error-prone SMS reads (PacBio SMRT or ONT) aligned to the reference
- Indexed FASTA reference genome the BAM was aligned to
- Candidate biallelic SNV sites (from Longshot pileup genotype likelihoods or a gzipped, tabix-indexed input VCF via --potential_variants)
- Filter parameters: min_mapq (default 20), min_allele_qual (default 7.0), max_cigar_indel (20), anchor_length (6), max_window (50), band_width (20)

## Outputs

- allelotype_observations.tsv: per-(read, SNV) table with read identifier, SNV position, called allele, and phred-scaled allele quality
- Optional per-step variant debug files when --variant_debug_dir is supplied
- Retained observations feed haplotype-informed phased genotyping and HP-tagged haplotype-separated BAM output

## How to apply

Load a sorted, indexed BAM/CRAM of long reads and an indexed FASTA reference, retaining for each candidate biallelic SNV only reads passing the min_mapq filter (default 20). For each read overlapping a candidate SNV, realign the read to indel-free ref and alt window versions (anchor_length 6, up to 50 padding bases via max_window) using the pair-HMM forward algorithm with band_width >= 20 to obtain p_ref and p_alt; discard the read-variant if its window contains a CIGAR I/D/N operation longer than 20 (max_cigar_indel). Assign the allele with the higher likelihood and convert the ref/alt posterior ratio to a phred-scaled allele quality, then drop observations with quality below 7.0 (min_allele_qual). Write retained observations to allelotype_observations.tsv; the quality cutoff reduces effective read depth for SMS reads but removes observations driven by sequencing error rather than true alleles.

## Related tools

- **Longshot** (Performs pair-HMM local realignment allelotyping of reads at candidate SNV sites and applies the min_mapq, max_cigar_indel, and min_allele_qual (7.0) filters to produce per-(read, SNV) allele observations) — https://github.com/pjedge/longshot
- **HapCUT2** (Downstream consumer context: Longshot uses HapCUT2 to estimate the most likely pair of haplotypes from the filtered allele observations during iterative phased genotyping)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf --min_mapq 20 --min_allele_qual 7.0
```

## Evaluation signals

- No retained observation in allelotype_observations.tsv has a phred-scaled allele quality below 7.0
- Every retained observation lists a read identifier, SNV position, called allele, and phred-scaled quality field
- No read with mapping quality below 20 (min_mapq default) contributes an allele observation
- Read-variant pairs whose realignment window contains a CIGAR I/D/N operation longer than 20 bp are absent from the output
- Observed per-read ref/alt likelihood ratios (p_ref vs p_alt from the pair-HMM forward algorithm) match the phred-scaled qualities in the table

## Limitations

- Allele quality filtering (phred < 7.0 discarded) reduces the effective read depth for SMS reads
- Accurate calling of short indels with SMS reads is challenging due to the high insertion/deletion error rate
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, affecting read-SNV assignments
- The pair-HMM forward algorithm can be swapped for max scoring alignment (-x) or numerically-stable logspace mode (-S); -S is significantly slower but tests showed this not to be necessary for highly error-prone reads such as PacBio CLR

## Evidence

- [discussion] Allele observations with phred-scaled quality score below a threshold (7.0 by default) are discarded. This reduces the effective read depth for SMS reads: [evidence span withheld — anchor status: not_found]
- [methods] min_mapq (minimum mapping quality filter): "Minimum mapping quality to use a read. [default: 20]"
- [discussion] Local realignment of reads to ref/alt sequences using pair-HMM: "This local realignment is performed using a pair-HMM29."
- [readme] Longshot allelotyping computes per-read allele likelihoods via the pair-HMM forward algorithm: "Longshot is a variant calling tool for diploid genomes using long error prone reads such as Pacific Biosciences (PacBio) SMRT and Oxford Nanopore Technologies (ONT)."
