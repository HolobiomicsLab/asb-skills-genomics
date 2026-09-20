---
name: multi-alignment-coverage-accounting
description: Use when contig sets from multiple assemblers (e.g., CLC bio, Newbler,
  SOAPdenovo, Velvet, IDBA, Ray, MeGAMerge MergedContigs.fasta) must be quality-checked
  via read-mapping coverage, especially when assembler-specific under-coverage is
  suspected.
when_to_use_negative:
- Input is raw untrimmed reads — reads must first be trimmed with the sliding window
  protocol (minimum quality score 2) before mapping
- The goal is assembly construction rather than post-assembly quality validation —
  use MeGAMerge itself instead
- Reference genomes are available for direct contig-to-genome alignment — use Nucmer-based
  reference genome coverage instead of read-mapping coverage
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: BWA
  role: Maps trimmed reads to each contig FASTA reference to produce SAM alignments
    used for coverage computation
- name: SAMtools
  role: Converts SAM to sorted indexed BAM files and parses them to compute per-reference
    percent coverage
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) whose coverage is compared
    against individual assembler contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:e279ab7a3c7c888679b28ba2a1357278d1670aa5527a591d073b35a7bcbfeae3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - BWA
  - MeGAMerge
  - SAMtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-alignment-coverage-accounting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates metagenome assembly quality by mapping trimmed reads back to contig FASTAs (individual assembler outputs and MeGAMerge output) with BWA, converting/sorting alignments to indexed BAM files with SAMtools, and computing per-contig and per-assembly percent coverage both excluding and including multiple alignments per read.

## When to use

Use when contig sets from multiple assemblers (e.g., CLC bio, Newbler, SOAPdenovo, Velvet, IDBA, Ray, MeGAMerge MergedContigs.fasta) must be quality-checked via read-mapping coverage, especially when assembler-specific under-coverage is suspected.

## When NOT to use

- Input is raw untrimmed reads — reads must first be trimmed with the sliding window protocol (minimum quality score 2) before mapping
- The goal is assembly construction rather than post-assembly quality validation — use MeGAMerge itself instead
- Reference genomes are available for direct contig-to-genome alignment — use Nucmer-based reference genome coverage instead of read-mapping coverage

## Inputs

- Contig FASTA files from individual assemblers (CLC bio, Newbler, SOAPdenovo, Velvet, IDBA, Ray)
- Final MeGAMerge contig FASTA (MergedContigs.fasta)
- Quality-trimmed sample reads (e.g., Horizon oil spill Illumina data) in FASTQ format

## Outputs

- SAM files from BWA read mappings per contig reference
- Sorted BAM files with associated BAM index files
- contig_coverage.csv aggregating per-contig and per-assembly percentage coverage (excluding and including multiple alignments per read)

## How to apply

Index each contig FASTA (the final MergedContigs.fasta and each individual assembler's contigs) as a BWA reference database, then map the quality-trimmed sample reads with BWA to produce SAM output. Convert and sort the alignments with SAMtools, generating sorted BAM files plus index files. Parse each BAM with SAMtools per-reference statistics to compute the percentage of contig bases covered by mapped reads, reporting values both excluding and including multiple alignments per read. Aggregate per-contig and per-assembly coverage into a single CSV table (contig_coverage.csv). Rationale: the paper uses this to expose that individual CLC bio assemblies of the Horizon oil spill sample are far less well covered (~95%, below the >=99% expectation) than MeGAMerged contigs, revealing assembler-specific weaknesses on complex metagenome samples.

## Related tools

- **BWA** (Maps trimmed reads to each contig FASTA reference to produce SAM alignments used for coverage computation)
- **SAMtools** (Converts SAM to sorted indexed BAM files and parses them to compute per-reference percent coverage)
- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) whose coverage is compared against individual assembler contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
bwa index MergedContigs.fasta; bwa mem MergedContigs.fasta reads_trimmed.fastq > aln.sam; samtools view -b aln.sam | samtools sort - aln_sorted; samtools index aln_sorted.bam; samtools idxstats aln_sorted.bam
```

## Evaluation signals

- CLC bio oil-spill individual assembly coverage falls near the reported ~85% (~88.2% when counting multiple alignments per read), below the >=99% expectation
- MeGAMerge contigs assembled without CLC bio achieve coverage near the reported ~99%
- Sorted BAM files exist for every contig reference and are accompanied by valid index files
- contig_coverage.csv contains per-contig rows and per-assembly aggregates with both exclusion and inclusion of multiple alignments per read
- MeGAMerged contigs show coverage supporting their validity relative to individual assembler contigs, consistent with read-mapping validation in the paper

## Limitations

- Coverage metrics count multiple alignments per read differently, so exclusion vs. inclusion of multiply-mapped reads can shift results substantially (e.g., 85% vs 88.2%)
- Read-mapping coverage is one of several validation approaches in the paper; it does not by itself assess chimeras or reference genome coverage (which requires Nucmer alignment)
- The paper's reported coverage values are for specific datasets (Horizon oil spill, MetaHit, HMP, simulated 1,000-genome data); thresholds may not generalize to other communities

## Evidence
