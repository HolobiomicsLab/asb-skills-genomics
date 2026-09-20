---
name: sam-bam-file-processing
description: Use when ONT nanopore reads (e.g. SQK-PCS111 cDNA) have been aligned
  with Minimap2 to a plasmid or mRNA reference and you need sorted, indexed BAMs with
  primary-only reads for QC and error analysis.
when_to_use_negative:
- Input is a short-read Illumina BAM processed with a different pipeline (this skill
  is grounded in Minimap2 map-ont ONT alignments)
- Reads have not been aligned yet — align with Minimap2 before SAM/BAM processing
- Secondary/alternative contig alignments are required, since the pipeline explicitly
  excludes them with -F 2048
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT nanopore reads to the plasmid reference using '-ax map-ont' to
    produce SAM input
- name: SAMtools v1.15
  role: Converts SAM to sorted and indexed BAM; extracts primary aligned reads with
    'SAMtools view -F 2048'
- name: pysamstats
  role: Computes per-nucleotide error profile of mapped reads relative to the reference
    from the BAM
  repo: https://github.com/alimanfoo/pysamstats
- name: Mana
  role: Consumes the ONT alignment BAM (-b flag) to generate automated mRNA quality
    reports
  repo: https://github.com/scchess/Mana
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:9d79dd700a7ff7768a47635b7f5ba67a88745747ea8c08029ed11307f920222d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alimanfoo/pysamstats
  tools:
  - BCFtools
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMTool
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sam-bam-file-processing

## Summary

Process SAM/BAM alignments from ONT nanopore reads of an mRNA vaccine plasmid reference: sort and index with SAMtools, extract primary alignments, and prepare the BAM for downstream QC (pysamstats error profiling, Mana quality reports, tailfindr poly(A) analysis).

## When to use

Use when ONT nanopore reads (e.g. SQK-PCS111 cDNA) have been aligned with Minimap2 to a plasmid or mRNA reference and you need sorted, indexed BAMs with primary-only reads for QC and error analysis.

## When NOT to use

- Input is a short-read Illumina BAM processed with a different pipeline (this skill is grounded in Minimap2 map-ont ONT alignments)
- Reads have not been aligned yet — align with Minimap2 before SAM/BAM processing
- Secondary/alternative contig alignments are required, since the pipeline explicitly excludes them with -F 2048

## Inputs

- SAM alignment file from Minimap2 '-ax map-ont' alignment of ONT nanopore reads
- Reference FASTA (plasmid carrying the eGFP mRNA vaccine sequence)
- Concatenated FASTQ of nanopore reads filtered at quality >9 (pre-alignment)

## Outputs

- Sorted and indexed BAM file (e.g. alignment.bam)
- Primary-aligned read subset (SAMtools view -F 2048) for read length distributions
- Position-wise statistics from pysamstats and QC reports from Mana

## How to apply

Align quality-filtered reads (retaining nanopore reads with quality scores >9) to the plasmid reference using Minimap2 Release 2.20-r1064 with the '-ax map-ont' preset for Nanopore data. Convert the resulting SAM to sorted, indexed BAM with SAMtools v1.15. Extract primary aligned reads excluding secondary alignments with 'SAMtools view -F 2048' before computing read length distributions or per-nucleotide statistics. The cleaned BAM then feeds downstream analyses: pysamstats (built on Pysam 0.20.0) for per-nucleotide mismatch/insertion/deletion profiles, bcftools v1.15 for variant calling and consensus generation, and Mana (docker image scchess/mana) for automated mRNA QC reports.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns ONT nanopore reads to the plasmid reference using '-ax map-ont' to produce SAM input)
- **SAMtools v1.15** (Converts SAM to sorted and indexed BAM; extracts primary aligned reads with 'SAMtools view -F 2048')
- **pysamstats** (Computes per-nucleotide error profile of mapped reads relative to the reference from the BAM) — https://github.com/alimanfoo/pysamstats
- **Mana** (Consumes the ONT alignment BAM (-b flag) to generate automated mRNA quality reports) — https://github.com/scchess/Mana

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- BAM index (.bai) file is generated alongside the sorted BAM without SAMtools errors
- 'SAMtools view -F 2048' yields primary alignments only (no secondary flags) for read length distribution
- Downstream pysamstats output shows per-nucleotide mismatch/insertion/deletion counts across the full mRNA length, including the template-encoded 3' poly(A) tail
- Mana accepts the BAM (-b /src/alignment.bam) and writes QC reports to the '-o' output directory within ~10 minutes
- Mean per-read error rate computed from processed alignments is ~5.0%, with poly(A) deletion rate ~13%, matching the reported cDNA profile

## Limitations

- Direct RNA sequencing alignments may be less reliable than cDNA; the authors found cDNA sequencing more reliable
- Default pysamstats pileup max depth is 8000, which may truncate statistics at very high coverage positions
- Data are proof of concept with small sample sizes (n=1 to n=4); alignment behaviour may vary on other samples

## Evidence

- [discussion] Minimap2 alignment with map-ont preset: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] SAMtools BAM generation and indexing: "processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] Primary alignment extraction flag: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [discussion] pysamstats per-nucleotide error profiling: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [readme] Mana README docker invocation consuming BAM: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam"
- [discussion] Quality filter of nanopore reads before alignment: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"
