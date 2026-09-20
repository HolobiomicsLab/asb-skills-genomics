---
name: cdna-sequencing-error-rate-analysis
description: Use when long-read cDNA nanopore (SQK-PCS111) reads are aligned to a
  known plasmid/mRNA reference and you need a per-read error rate and per-nucleotide
  mismatch/insertion/deletion profile, e.g. characterising error in a poly(A) tail.
when_to_use_negative:
- The goal is to call true sequence variants or build a consensus — this skill quantifies
  the sequencing error background, not biology; use variant calling/consensus generation
  (e.g. bcftools v1.15) instead.
- Input data is short-read Illumina — this pipeline's Minimap2 '-ax map-ont' preset
  and nanopore error modelling are specific to ONT cDNA reads.
- Reads have not passed the Q>9 quality filter — including low-quality nanopore reads
  will inflate the error profile beyond the reported baseline.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3569
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns quality-filtered nanopore reads to the plasmid reference using the
    '-ax map-ont' preset for Nanopore data
- name: SAMtools v1.15
  role: Converts SAM alignments to sorted and indexed BAM files and extracts primary
    aligned reads with 'SAMtools view -F 2048'
- name: pysamstats
  role: Computes the per-nucleotide error profile (mismatches, insertions, deletions)
    of mapped reads relative to the reference sequence
  repo: https://github.com/alimanfoo/pysamstats
- name: Mana
  role: Analyses ONT alignments (.BAM) to report mRNA quality control statistics supporting
    VAX-seq
  repo: https://github.com/scchess/Mana
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:f1284514a04eeb7893b92536ca81bcc0aeee093a87b375c7c31bd9e496538d23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alimanfoo/pysamstats
  tools:
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cdna-sequencing-error-rate-analysis

## Summary

Quantifies the per-read sequencing error rate and per-nucleotide mismatch/insertion/deletion profile of ONT cDNA nanopore reads (SQK-PCS111) aligned to a reference plasmid, including the elevated deletion rate within a template-encoded 3' poly(A) tail. It establishes the baseline error profile needed to distinguish sequencing error from true sequence variation in mRNA vaccine QC.

## When to use

Use when long-read cDNA nanopore (SQK-PCS111) reads are aligned to a known plasmid/mRNA reference and you need a per-read error rate and per-nucleotide mismatch/insertion/deletion profile, e.g. characterising error in a poly(A) tail.

## When NOT to use

- The goal is to call true sequence variants or build a consensus — this skill quantifies the sequencing error background, not biology; use variant calling/consensus generation (e.g. bcftools v1.15) instead.
- Input data is short-read Illumina — this pipeline's Minimap2 '-ax map-ont' preset and nanopore error modelling are specific to ONT cDNA reads.
- Reads have not passed the Q>9 quality filter — including low-quality nanopore reads will inflate the error profile beyond the reported baseline.

## Inputs

- SQK-PCS111 cDNA nanopore FASTQ reads with per-read quality scores
- Plasmid/reference FASTA containing the eGFP mRNA construct sequence
- Sorted and indexed BAM alignment of filtered reads to the plasmid reference

## Outputs

- Per-read mean error rate over primary aligned reads (expect ~5.0%)
- Per-nucleotide mismatch/insertion/deletion counts relative to the plasmid reference (pysamstats output)
- Deletion rate within the template-encoded 3' poly(A) tail (expect ~13%)

## How to apply

Filter SQK-PCS111 cDNA nanopore reads retaining only quality scores >9 to build a concatenated FASTQ. Align to the plasmid reference with Minimap2 (Release 2.20-r1064) using the '-ax map-ont' preset, then process the SAM with SAMtools v1.15 to produce sorted, indexed BAMs and extract primary alignments via 'samtools view -F 2048'. Use pysamstats (built on Pysam 0.20.0) against the reference index sequence to obtain per-nucleotide match/mismatch/deletion/insertion counts (e.g. the 'variation' statistics type documented in its README), and aggregate these to a per-read mean error rate. Separately aggregate error counts over the template-encoded 3' poly(A) tail region to quantify its deletion rate. Validation: the mean per-read error rate should be ~5.0% and the poly(A) tail deletion rate elevated at ~13% for the reference eGFP mRNA.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns quality-filtered nanopore reads to the plasmid reference using the '-ax map-ont' preset for Nanopore data)
- **SAMtools v1.15** (Converts SAM alignments to sorted and indexed BAM files and extracts primary aligned reads with 'SAMtools view -F 2048')
- **pysamstats** (Computes the per-nucleotide error profile (mismatches, insertions, deletions) of mapped reads relative to the reference sequence) — https://github.com/alimanfoo/pysamstats
- **Mana** (Analyses ONT alignments (.BAM) to report mRNA quality control statistics supporting VAX-seq) — https://github.com/scchess/Mana

## Examples

```
samtools view -F 2048 sorted.bam > primary.bam; pysamstats --type variation --fasta reference.fasta primary.bam > error_profile.tsv
```

## Evaluation signals

- Mean per-read error rate across primary alignments is ~5.0% for the reference eGFP mRNA
- Deletion rate aggregated over the template-encoded 3' poly(A) tail is elevated at ~13% relative to the transcript body
- All reads entering the profile passed the quality score >9 filter, and secondary alignments (flag 2048) were excluded
- BAM is sorted and indexed, and pysamstats per-position counts sum consistently with primary-read coverage across the mRNA

## Limitations

- Nanopore cDNA reads carry an intrinsic ~5% error rate, so per-base error must be modelled before interpreting low-frequency variants
- Deletion errors are markedly enriched in homopolymer/poly(A) regions (~13% in the poly(A) tail), so exact poly(A) length cannot be resolved per-read from cDNA error profiles alone
- Reads were limited to quality scores >9; unfiltered or direct RNA data may show a different error profile, and data are proof of concept (n = 1 to n = 4)

## Evidence

- [discussion] Filter steps: Quality filtering of nanopore reads at Q>9: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"
- [discussion] Read alignment to plasmid reference with Minimap2: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] Per-nucleotide error profile determination: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] Primary alignment extraction: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [readme] pysamstats variation statistics capability: [evidence span withheld — anchor status: external_doc]
