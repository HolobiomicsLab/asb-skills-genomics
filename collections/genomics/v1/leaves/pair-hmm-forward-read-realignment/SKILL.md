---
name: pair-hmm-forward-read-realignment
description: Use when you need per-read allele likelihoods (p_ref vs p_alt) at candidate
  SNVs in long error-prone SMS reads (PacBio/ONT), e.g. as the allelotyping stage
  before haplotype-informed genotyping.
when_to_use_negative:
- Your reads are short Illumina reads — the paper positions Longshot as a method leveraging
  long SMS reads where haplotype information is present
- The variant is a structural variant longer than 50 bp — Longshot does not support
  SVs (only SNVs; indels are genotyped only if given in an input VCF)
- You need a single best alignment per read rather than likelihoods averaged over
  all alignments — the -x/--max_alignment flag switches away from the pair-HMM forward
  algorithm, indicating the pair-HMM is not the right default for max-scoring alignment
  use cases
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Implements the pair-HMM forward local realignment and allelotyping of reads
    at candidate SNV sites
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:22cd18f869196aa32ba94b58b5401398b490d717b866e284e4a407a81d7b82f4
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# pair-hmm-forward-read-realignment

## Summary

Allelotyping of individual long reads at candidate biallelic SNV sites by realigning each read to reference and alternate windows using a pair-HMM forward algorithm, yielding per-read allele assignments and phred-scaled allele quality scores.

## When to use

Use when you need per-read allele likelihoods (p_ref vs p_alt) at candidate SNVs in long error-prone SMS reads (PacBio/ONT), e.g. as the allelotyping stage before haplotype-informed genotyping.

## When NOT to use

- Your reads are short Illumina reads — the paper positions Longshot as a method leveraging long SMS reads where haplotype information is present
- The variant is a structural variant longer than 50 bp — Longshot does not support SVs (only SNVs; indels are genotyped only if given in an input VCF)
- You need a single best alignment per read rather than likelihoods averaged over all alignments — the -x/--max_alignment flag switches away from the pair-HMM forward algorithm, indicating the pair-HMM is not the right default for max-scoring alignment use cases

## Inputs

- Sorted, indexed BAM/CRAM file of long SMS reads (PacBio SMRT or Oxford Nanopore) aligned to the reference
- Indexed FASTA reference genome that the BAM is aligned to
- Candidate biallelic SNV list (VCF, gzipped and tabix-indexed, e.g. via --potential_variants)

## Outputs

- Per-(read, SNV) allele observations with read identifier, SNV position, called allele, and phred-scaled allele quality (allelotype_observations.tsv)
- Per-read allele assignment used downstream for haplotype-informed genotyping and HP tags in the haplotype-separated BAM

## How to apply

Load a sorted, indexed BAM/CRAM of long reads and an indexed FASTA reference, retaining for each candidate biallelic SNV only reads passing the min_mapq filter (default 20). For each read-SNV pair, build a realignment window with indel-free anchor sequences of length 6 (anchor_length) on both sides and up to 50 padding bases (max_window) around the variant, then realign the read to the reference and alternate window versions using the pair-HMM forward algorithm with a minimum band width of 20 (band_width) to obtain p_ref and p_alt. Discard read-variant observations whose window contains a CIGAR indel (I/D/N) longer than 20 (max_cigar_indel), assign the read to the more likely allele, and convert the ref/alt posterior ratio to a phred-scaled allele quality; drop observations with quality below 7.0 (min_allele_qual). The resulting per-(read, SNV) allele observations reduce noise from the high SMS error rate by averaging over all possible local alignments rather than using a single max-scoring alignment.

## Related tools

- **Longshot** (Implements the pair-HMM forward local realignment and allelotyping of reads at candidate SNV sites) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Output allelotype_observations.tsv contains read identifier, SNV position, called allele, and phred-scaled quality fields
- No retained allele observation has a phred-scaled quality below the default threshold of 7.0
- No read-variant observation retained with a window CIGAR indel (I/D/N) longer than max_cigar_indel = 20
- On the README example dataset, called variants match example_data/ground_truth_variants.vcf with high accuracy (714 true variants on 3 simulated 200 kb contigs)
- Per-read assigned alleles are consistent with the haplotype tags (HP:i:1 and HP:i:2) written to the output BAM

## Limitations

- Allele quality filtering (default 7.0) reduces the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- Accurate short indel calling with SMS reads is challenging due to the high insertion/deletion error rate
- Excessive false potential variants in the input VCF can lead to inaccurate results — every variant is used and only allele fields are considered

## Evidence

- [discussion] pair-HMM forward averaging over local alignments: [evidence span withheld — anchor status: not_found]
- [discussion] min_allele_qual threshold: [evidence span withheld — anchor status: not_found]
- [discussion] local realignment via pair-HMM: "This local realignment is performed using a pair-HMM29."
- [methods] min_mapq default: "Minimum mapping quality to use a read. [default: 20]"
- [readme] stable alignment option: "Use numerically-stable (logspace) pair HMM forward algorithm. Is significantly slower but may be more accurate."
