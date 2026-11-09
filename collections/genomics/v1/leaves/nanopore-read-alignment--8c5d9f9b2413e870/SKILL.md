---
name: nanopore-read-alignment
description: Use when ONT long-read FASTQ data (e.g. SQK-PCS111 cDNA, SQK-RNA002 dRNA,
  or SQK-LSK109 pDNA reads) must be mapped to a plasmid or mRNA reference as the first
  step of VAX-seq quality analysis.
when_to_use_negative:
- Input is Illumina short-read data, which the article handles separately with TrimGalore
  Q20 trimming rather than the map-ont preset
- Only per-base quality statistics are needed and reads are already provided as an
  indexed BAM — go directly to Mana or pysamstats instead of re-aligning
- The goal is modified-nucleoside (N1-methylpseudouridine) detection via direct RNA
  error signatures without a matching reference to align against
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT reads to the plasmid or mRNA reference using the -ax map-ont preset;
    also realigns unmapped reads to an E. coli reference for contamination quantification
- name: SAMtools v1.15
  role: Converts Minimap2 SAM output into sorted, indexed BAM files and extracts primary
    aligned reads with 'view -F 2048'
- name: Mana
  role: Consumes the sorted alignment BAM with the reference FASTA to produce automated
    mRNA QC reports (error profiles, length distribution, integrity)
  repo: https://github.com/scchess/Mana
- name: pysamstats (Pysam 0.20.0)
  role: Computes per-nucleotide error profiles of mapped reads relative to the reference
    index sequence from the BAM
  repo: https://github.com/alimanfoo/pysamstats
- name: Porechop v0.2.4
  role: Removes barcodes from ONT reads prior to alignment
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  - task_004
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:39224e3d870c96104c4bc16b8d92afd7960a6a227ea773c1fb31ab1bf83a64fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Oxford Nanopore Technologies (ONT)
  - Porechop v0.2.4
  - Pysam
  - SAMTool
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  - pysamstats (Pysam 0.20.0)
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nanopore-read-alignment

## Summary

Align ONT nanopore reads (cDNA, direct RNA, or plasmid template) to a reference sequence using Minimap2 with the -ax map-ont preset, then process SAM output into sorted, indexed BAM files via SAMtools for downstream mRNA QC (pysamstats error profiling, Mana reporting, contamination quantification).

## When to use

Use when ONT long-read FASTQ data (e.g. SQK-PCS111 cDNA, SQK-RNA002 dRNA, or SQK-LSK109 pDNA reads) must be mapped to a plasmid or mRNA reference as the first step of VAX-seq quality analysis.

## When NOT to use

- Input is Illumina short-read data, which the article handles separately with TrimGalore Q20 trimming rather than the map-ont preset
- Only per-base quality statistics are needed and reads are already provided as an indexed BAM — go directly to Mana or pysamstats instead of re-aligning
- The goal is modified-nucleoside (N1-methylpseudouridine) detection via direct RNA error signatures without a matching reference to align against

## Inputs

- ONT nanopore FASTQ reads (cDNA SQK-PCS111, direct RNA SQK-RNA002, or pDNA SQK-LSK109)
- Reference sequence FASTA (e.g. linearised eGFP plasmid reference or mRNA transcript)
- E. coli reference FASTA (for unmapped-read contamination realignment)

## Outputs

- Sorted and indexed BAM alignment file (via SAMtools v1.15)
- SAM alignment output from Minimap2
- FASTQ/FASTA of unmapped reads for secondary contamination alignment
- Primary-aligned BAM (SAMtools view -F 2048) for length-distribution and error analysis

## How to apply

Filter nanopore reads to quality scores >9 to form a concatenated FASTQ, then align with Minimap2 (Release 2.20-r1064) using the '-ax map-ont' preset tuned for Nanopore reads. Convert the SAM output to sorted and indexed BAM files with SAMtools v1.15. For mRNA integrity analysis, extract primary aligned reads only, excluding secondary alignments, with 'SAMtools view -F 2048' before computing read-length distributions. For contamination assessment, extract unmapped reads and realign them to an E. coli reference with Minimap2 to quantify bacterial carry-over. The resulting BAM plus reference FASTA can then be passed to Mana (docker run --mrna -b alignment.bam -f file.fasta -o out) or to pysamstats for per-nucleotide error profiling.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns ONT reads to the plasmid or mRNA reference using the -ax map-ont preset; also realigns unmapped reads to an E. coli reference for contamination quantification)
- **SAMtools v1.15** (Converts Minimap2 SAM output into sorted, indexed BAM files and extracts primary aligned reads with 'view -F 2048')
- **Mana** (Consumes the sorted alignment BAM with the reference FASTA to produce automated mRNA QC reports (error profiles, length distribution, integrity)) — https://github.com/scchess/Mana
- **pysamstats (Pysam 0.20.0)** (Computes per-nucleotide error profiles of mapped reads relative to the reference index sequence from the BAM) — https://github.com/alimanfoo/pysamstats
- **Porechop v0.2.4** (Removes barcodes from ONT reads prior to alignment)

## Examples

```
minimap2 -ax map-ont plasmid_reference.fasta reads.fastq | samtools sort -o alignment.bam && samtools index alignment.bam
```

## Evaluation signals

- BAM file is sorted and indexed without errors after SAMtools processing
- For linearised eGFP pDNA (SQK-LSK109), read-fate fractions reproduce the reported 86.8% plasmid-aligned, 6.8% E. coli, 6.6% failed reads (summing to ~100%)
- After 'samtools view -F 2048', no secondary alignments remain in the extracted BAM used for length-distribution analysis
- Coverage over the eGFP mRNA is full and homogeneous, consistent with cDNA sequencing confirming sequence identity across the entire vaccine length
- Per-nucleotide error statistics computed by pysamstats exclude non-primary reads and, for dRNA analysis, the poly(A) tail region

## Limitations

- Minimap2 release is pinned to 2.20-r1064 and SAMtools to v1.15; other versions were not validated in this workflow
- The custom pipeline's released code availability is not guaranteed — the article notes pDNA data were 'mapped and analysed using a custom pipeline' whose public release is not stated
- cDNA sequencing was found more reliable than direct RNA sequencing; dRNA alignments carry dRNA-specific per-nucleotide error biases (e.g. deletion of low-quality poly(A) nucleotides)

## Evidence

- [discussion] aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] SAMtools sorted indexed BAM: "The resulting SAM alignment files were processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] quality filter Q>9: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"
- [readme] Mana CLI purpose: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [discussion] unmapped reads to E. coli: "This FASTA file was aligned to an E.coli reference sequence utilising Minimap2"
