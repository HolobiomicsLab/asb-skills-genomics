---
name: short-read-vs-long-read-coverage-comparison
description: Use when both Illumina and ONT cDNA BAM alignments of the same mRNA (e.g.
  eGFP vaccine construct) are available and you must quantify short-read misalignment
  at low-complexity poly(A) regions versus long-read coverage.
when_to_use_negative:
- Only one sequencing platform's BAM is available — the comparison requires paired
  Illumina and ONT cDNA alignments of the same construct.
- The goal is direct RNA chemistry analysis (e.g. N1-methylpseudouridine detection);
  this skill addresses cDNA/short-read coverage comparison, not nucleoside modification
  calling.
- Input is raw FASTQ without prior alignment; Mana operates on alignment BAM files,
  so reads must first be barcode-trimmed (Porechop), aligned (Minimap2), and sorted/indexed
  (SAMtools).
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: Mana
  role: Orchestrates the --mrna analysis, internally calling SAMtools, BCFtools, Pysam
    and bedtools to produce coverage, error-profile, and consensus statistics from
    each BAM
  repo: https://github.com/scchess/Mana
- name: SAMtools
  role: Generates sorted and indexed BAM files used as Mana input
- name: BCFtools
  role: Variant calling and consensus sequence generation for consensus accuracy assessment
- name: Pysam
  role: Per-nucleotide error profile of mapped reads determined relative to the reference
    via pysamstats
  repo: https://github.com/alimanfoo/pysamstats
- name: Minimap2
  role: Aligned ONT reads to the plasmid reference with -ax map-ont to produce the
    long-read BAM compared here
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read cDNA sequencing platform whose uniform coverage is benchmarked against
    Illumina
- name: Illumina
  role: Short-read TruSeq sequencing platform whose poly(A) tail misalignment is quantified
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:3df2f7e5784c69491003b4de1390eef602a68d8b77be981c794c6c681d2c67a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Illumina
  - Mana
  - Minimap2
  - Oxford Nanopore Technologies (ONT)
  - Pysam
  - SAMtools
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# short-read-vs-long-read-coverage-comparison

## Summary

Compares Illumina TruSeq short-read and ONT cDNA long-read sequencing performance over an mRNA vaccine sequence, focusing on per-nucleotide coverage homogeneity and consensus accuracy at the template-encoded 3' poly(A) tail, using the Mana toolkit (SAMtools, BCFtools, Pysam/pysamstats, bedtools).

## When to use

Use when both Illumina and ONT cDNA BAM alignments of the same mRNA (e.g. eGFP vaccine construct) are available and you must quantify short-read misalignment at low-complexity poly(A) regions versus long-read coverage.

## When NOT to use

- Only one sequencing platform's BAM is available — the comparison requires paired Illumina and ONT cDNA alignments of the same construct.
- The goal is direct RNA chemistry analysis (e.g. N1-methylpseudouridine detection); this skill addresses cDNA/short-read coverage comparison, not nucleoside modification calling.
- Input is raw FASTQ without prior alignment; Mana operates on alignment BAM files, so reads must first be barcode-trimmed (Porechop), aligned (Minimap2), and sorted/indexed (SAMtools).

## Inputs

- Illumina TruSeq alignment BAM (Q20-trimmed FASTQ reads aligned to the reference)
- ONT cDNA alignment BAM (Minimap2 -ax map-ont aligned, quality >9 reads)
- Reference FASTA of the mRNA/plasmid construct with template-encoded 3' poly(A) tail

## Outputs

- Per-nucleotide coverage profiles over the full mRNA for both alignments
- Per-nucleotide error profiles over the 3' poly(A) tail region (from pysamstats)
- Consensus accuracy and consensus sequence statistics (BCFtools variant calling)
- Coverage homogeneity statistics and Illumina replicate squared correlation (R2 ≈ 0.99)

## How to apply

Mount the Illumina TruSeq alignment BAM, the ONT cDNA alignment BAM, and the reference FASTA into the Mana docker image with -v ${PWD}:/src, then run 'python3 mana.py --mrna' twice, once with -b pointing to each BAM, letting Mana invoke SAMtools, BCFtools, Pysam/pysamstats and bedtools to generate per-nucleotide coverage, error-profile, and consensus statistics. Extract per-nucleotide coverage and error profiles over the 3' poly(A) tail region from both runs and quantify Illumina misalignment and the drop in consensus accuracy relative to the reference-encoded poly(A) sequence. Compute coverage homogeneity statistics showing ONT cDNA reads cover the entire mRNA length uniformly while Illumina coverage is heterogeneous and disrupted at the poly(A) tail. Finally, correlate the Illumina coverage profiles of sequencing replicates and verify the squared correlation reaches the reported R2 ≈ 0.99.

## Related tools

- **Mana** (Orchestrates the --mrna analysis, internally calling SAMtools, BCFtools, Pysam and bedtools to produce coverage, error-profile, and consensus statistics from each BAM) — https://github.com/scchess/Mana
- **SAMtools** (Generates sorted and indexed BAM files used as Mana input)
- **BCFtools** (Variant calling and consensus sequence generation for consensus accuracy assessment)
- **Pysam** (Per-nucleotide error profile of mapped reads determined relative to the reference via pysamstats) — https://github.com/alimanfoo/pysamstats
- **Minimap2** (Aligned ONT reads to the plasmid reference with -ax map-ont to produce the long-read BAM compared here)
- **Oxford Nanopore Technologies (ONT)** (Long-read cDNA sequencing platform whose uniform coverage is benchmarked against Illumina)
- **Illumina** (Short-read TruSeq sequencing platform whose poly(A) tail misalignment is quantified)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/illumina.bam -f /src/reference.fasta -o /src/out
```

## Evaluation signals

- Mana completes both --mrna runs (Illumina BAM and ONT BAM) without error (~10 min per run per README).
- Per-nucleotide error profile over the poly(A) tail shows elevated Illumina errors and poor consensus accuracy versus the ONT run.
- Coverage homogeneity statistics show ONT cDNA coverage is full and homogeneous across the entire mRNA length, while Illumina coverage is heterogeneous and disrupted at the poly(A) tail.
- Squared correlation of Illumina replicate coverage profiles reaches approximately R2 = 0.99.
- Output directory contains per-nucleotide coverage and error-profile files and consensus statistics for both alignments.

## Limitations

- Short-read (Illumina) misalignment at the low-complexity poly(A) tail causes many errors and poor consensus accuracy, so Illumina consensus over this region should not be trusted.
- Illumina coverage is heterogeneous across the mRNA, so cross-platform coverage comparisons are confounded outside the ONT benchmark.
- Mana analysis requires docker with correctly mounted paths (-v ${PWD}:/src); incorrect mounting silently breaks file access.

## Evidence

- [results] Short-read misalignment at the poly(A) tail caused many errors and poor consensus accuracy: [evidence span withheld — anchor status: not_found]
- [discussion] Nanopore cDNA coverage of entire mRNA vaccine length: "Nanopore cDNA sequencing was sufficient to confirm sequence identity with full and homogenous coverage of the entire mRNA vaccine length"
- [discussion] Per-nucleotide error profile with pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [readme] Mana docker invocation: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta \
               -o /src/out"
- [discussion] Variant calling with bcftools: "Variant calling and consensus sequence generation was completed using bcftools (v1.15)"
