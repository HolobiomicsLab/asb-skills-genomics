---
name: phred-scaled-allele-quality-computation
description: Use when calling SNVs from error-prone PacBio SMRT or ONT long reads
  with Longshot and per-(read, SNV) allele observations with phred-scaled allele quality
  are needed for haplotype-informed genotyping.
when_to_use_negative:
- Input reads are short Illumina reads — Longshot allelotyping is designed for long
  error-prone SMS reads where haplotype information spans the variant
- The variants of interest are indels or structural variants (length > 50 bp) — Longshot
  only calls SNVs and does not support structural variants
- You need called variants only, not per-read allele assignments — run the standard
  Longshot variant-calling command without the allelotype-observation output
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Performs pair-HMM local realignment allelotyping of reads at candidate SNV
    sites and emits phred-scaled allele qualities
  repo: github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:2e3350f9b0cc4a11ff85aae56ae8d4ee55caaee18ed86e133b2280c8f610395c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# phred-scaled-allele-quality-computation

## Summary

Longshot's allelotyping step assigns each read overlapping a candidate biallelic SNV to the ref or alt allele via pair-HMM forward-algorithm likelihoods, then converts the ref/alt posterior ratio into a phred-scaled allele quality so low-confidence read observations (default < 7.0) are discarded before genotyping.

## When to use

Use when calling SNVs from error-prone PacBio SMRT or ONT long reads with Longshot and per-(read, SNV) allele observations with phred-scaled allele quality are needed for haplotype-informed genotyping.

## When NOT to use

- Input reads are short Illumina reads — Longshot allelotyping is designed for long error-prone SMS reads where haplotype information spans the variant
- The variants of interest are indels or structural variants (length > 50 bp) — Longshot only calls SNVs and does not support structural variants
- You need called variants only, not per-read allele assignments — run the standard Longshot variant-calling command without the allelotype-observation output

## Inputs

- sorted, indexed BAM/CRAM file of long SMS reads (PacBio SMRT or Oxford Nanopore) aligned to the reference
- indexed FASTA reference genome that the BAM is aligned to
- candidate biallelic SNV sites (from Longshot pileup genotype likelihoods or a gzipped, tabix-indexed VCF passed via --potential_variants)

## Outputs

- allelotype_observations.tsv with read identifier, SNV position, called allele, and phred-scaled allele quality per (read, SNV) pair
- retained allele observations (all with phred-scaled quality >= 7.0) consumed downstream by haplotype-informed phased genotyping
- optional per-step variant debug files when --variant_debug_dir is supplied

## How to apply

Load the sorted, indexed BAM of long reads and indexed FASTA reference, retaining only reads passing the min_mapq filter (default 20). For each read-SNV pair, build the realignment window using indel-free anchors of length 6 (anchor_length) on each side plus up to 50 padding bases (max_window), then realign the read to the reference and alternate window versions using the pair-HMM forward algorithm with a minimum band width of 20 to obtain p_ref and p_alt. Discard a read-variant if its window contains a CIGAR indel (I/D/N) longer than 20 (max_cigar_indel), assign the read to the more likely allele, and convert the ref/alt posterior ratio to a phred-scaled allele quality. Drop allele observations below the min_allele_qual default of 7.0, since this quality filter reduces the effective read depth for SMS reads, and write per-(read, SNV) observations to allelotype_observations.tsv. Optionally use --stable_alignment (logspace pair-HMM forward algorithm), which is slower but may be more accurate.

## Related tools

- **Longshot** (Performs pair-HMM local realignment allelotyping of reads at candidate SNV sites and emits phred-scaled allele qualities) — github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- allelotype_observations.tsv exists and contains read identifier, SNV position, called allele, and phred-scaled quality fields
- no retained allele observation has phred-scaled allele quality below the 7.0 default threshold
- each retained observation's called allele matches the allele (ref or alt) with the higher pair-HMM likelihood (p_ref vs p_alt)
- read-variant pairs whose realignment window contains a CIGAR indel (I/D/N) longer than 20 are absent from the output
- all retained reads pass the min_mapq default of 20

## Limitations

- Allele quality filtering at 7.0 reduces the effective read depth for SMS reads
- The numerically-stable (logspace) pair-HMM forward algorithm (--stable_alignment) is significantly slower, though tests showed it not to be necessary for highly error-prone PacBio CLR reads
- Only biallelic SNV allelotyping is supported; indel genotyping requires an input VCF and phasing of indels is not performed

## Evidence

- [discussion] Allele observations with phred-scaled quality score below a threshold (7.0 by default) are discarded. This reduces the effective read depth for SMS reads: [evidence span withheld — anchor status: not_found]
- [discussion] local realignment pair-HMM: "This local realignment is performed using a pair-HMM29."
- [readme] min_mapq default 20: "Minimum mapping quality to use a read. [default: 20]"
- [readme] stable alignment flag: "-S, --stable_alignment    Use numerically-stable (logspace) pair HMM forward algorithm. Is significantly slower but"
- [discussion] haplotype-informed genotyping depends on allelotyping: "heterozygous SNVs can be called accurately in diploid genomes, by combining sensitive allelotyping of reads at SNV sites with haplotype-informed genotyping."
