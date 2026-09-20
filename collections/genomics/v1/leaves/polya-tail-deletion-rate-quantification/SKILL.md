---
name: polya-tail-deletion-rate-quantification
description: Use when analysing ONT cDNA nanopore reads (e.g. SQK-PCS111) aligned
  to a reference mRNA vaccine and you need per-read error rates or mismatch/insertion/deletion
  counts, especially to assess deletions in the 3' poly(A) tail.
when_to_use_negative:
- Input alignments are from direct RNA sequencing or short-read (Illumina) data rather
  than ONT cDNA nanopore reads, since the '-ax map-ont' preset and error profile are
  cDNA-nanopore-specific
- The goal is consensus/sequence-identity calling or variant detection rather than
  per-read per-nucleotide error profiling — use bcftools-based variant calling/consensus
  generation instead
- No template-encoded poly(A) tail region exists in the reference, so the headline
  poly(A) deletion-rate metric is not applicable
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns quality-filtered ONT cDNA reads to the plasmid reference using the
    '-ax map-ont' Nanopore preset to produce SAM alignments
- name: SAMtools v1.15
  role: Converts SAM to sorted/indexed BAM and extracts primary aligned reads via
    'SAMtools view -F 2048' prior to error profiling
- name: pysamstats
  role: Computes the per-nucleotide error profile (matches, mismatches, deletions,
    insertions) of mapped reads relative to the reference, built on Pysam 0.20.0
  repo: https://github.com/alimanfoo/pysamstats
- name: Mana
  role: Command-line toolkit that analyses ONT alignments (.BAM) to report automated
    mRNA quality control statistics
  repo: https://github.com/scchess/Mana
- name: Porechop v0.2.4
  role: Removes barcodes from ONT reads before alignment and error profiling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:5d91d8677670d7820a96f6b759811c879c444206644f5e990abd433a045e67cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alimanfoo/pysamstats
  tools:
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Porechop v0.2.4
  - Pysam
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# polya-tail-deletion-rate-quantification

## Summary

Quantifies the per-nucleotide error profile—particularly the elevated deletion rate within a template-encoded 3' poly(A) tail—of ONT cDNA nanopore reads aligned to a reference mRNA sequence, as demonstrated for an eGFP mRNA vaccine (mean 5.0% per-read error, ~13% deletion rate in the poly(A) tail).

## When to use

Use when analysing ONT cDNA nanopore reads (e.g. SQK-PCS111) aligned to a reference mRNA vaccine and you need per-read error rates or mismatch/insertion/deletion counts, especially to assess deletions in the 3' poly(A) tail.

## When NOT to use

- Input alignments are from direct RNA sequencing or short-read (Illumina) data rather than ONT cDNA nanopore reads, since the '-ax map-ont' preset and error profile are cDNA-nanopore-specific
- The goal is consensus/sequence-identity calling or variant detection rather than per-read per-nucleotide error profiling — use bcftools-based variant calling/consensus generation instead
- No template-encoded poly(A) tail region exists in the reference, so the headline poly(A) deletion-rate metric is not applicable

## Inputs

- ONT cDNA nanopore reads in FASTQ format (e.g. SQK-PCS111 runs of an eGFP mRNA vaccine)
- Plasmid reference sequence containing the template-encoded 3' poly(A) tail (FASTA)
- Sorted and indexed BAM file of primary alignments to the plasmid reference

## Outputs

- Per-read mean error rate across mapped reads (~5.0% expected for this data)
- Per-nucleotide mismatch, insertion, and deletion counts relative to the plasmid reference (from pysamstats variation stats)
- Deletion rate quantified within the template-encoded 3' poly(A) tail (~13% expected)

## How to apply

Filter ONT cDNA reads retaining only those with quality scores >9 into a concatenated FASTQ, then align to the plasmid reference with Minimap2 (Release 2.20-r1064) using the '-ax map-ont' preset. Process the SAM output with SAMtools v1.15 to generate sorted and indexed BAM files, and extract primary alignments with 'SAMtools view -F 2048' to exclude secondary alignments. Determine the per-nucleotide error profile of mapped reads relative to the reference using pysamstats (built on Pysam 0.20.0), using its variation statistics to obtain per-position match, mismatch, deletion, and insertion counts. Aggregate per-read mean error rates and per-nucleotide mismatch/insertion/deletion counts, then compute the deletion rate within the template-encoded 3' poly(A) tail region. The rationale is that the poly(A) tail is homopolymeric and prone to length estimation errors, so deletion rates there diagnose sequencing reliability rather than true sequence variation.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns quality-filtered ONT cDNA reads to the plasmid reference using the '-ax map-ont' Nanopore preset to produce SAM alignments)
- **SAMtools v1.15** (Converts SAM to sorted/indexed BAM and extracts primary aligned reads via 'SAMtools view -F 2048' prior to error profiling)
- **pysamstats** (Computes the per-nucleotide error profile (matches, mismatches, deletions, insertions) of mapped reads relative to the reference, built on Pysam 0.20.0) — https://github.com/alimanfoo/pysamstats
- **Mana** (Command-line toolkit that analyses ONT alignments (.BAM) to report automated mRNA quality control statistics) — https://github.com/scchess/Mana
- **Porechop v0.2.4** (Removes barcodes from ONT reads before alignment and error profiling)

## Examples

```
pysamstats --type variation --fasta reference.fasta alignment.bam > error_profile.tsv
```

## Evaluation signals

- Computed mean per-read error rate should be ~5.0% for the SQK-PCS111 cDNA eGFP dataset after Q>9 filtering
- Deletion rate within the template-encoded 3' poly(A) tail should be elevated at ~13%, above the genome-wide average deletion rate
- BAM files must be sorted and indexed, and contain only primary alignments (no secondary alignments, i.e. -F 2048 filter applied)
- Coverage over the entire mRNA vaccine length should be full and homogeneous, confirming sequence identity before interpreting error profiles
- pysamstats variation output must contain per-position match, mismatch, deletion, and insertion counts spanning the reference coordinates

## Limitations

- Data are proof of concept, ranging from n = 1 to n = 4 samples, so the error profile may not generalise across runs or platforms
- Errors are concentrated in homopolymeric regions — the elevated ~13% deletion rate in the poly(A) tail limits precise tail-length measurement from per-base error profiles alone
- Experiments were not randomised and investigators were not blinded, as template quality is the most significant determinant of sequencing quality

## Evidence

- [results] individual reads showing a mean 5.0% error rate and a higher deletion rate of 13% in the poly(A) tail: [evidence span withheld — anchor status: not_found]
- [discussion] alignment with Minimap2 -ax map-ont: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] per-nucleotide error profile determined with pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] Q>9 read filtering: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"
- [readme] pysamstats variation stats description: [evidence span withheld — anchor status: external_doc]
