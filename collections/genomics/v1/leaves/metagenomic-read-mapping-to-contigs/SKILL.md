---
name: metagenomic-read-mapping-to-contigs
description: 'Use when you need to judge assembly quality by read support: you have
  assembled contig FASTA files (individual or merged) and matching trimmed reads,
  and want per-contig base-coverage percentages.'
when_to_use_negative:
- Input is raw reads without assembled contigs — this skill validates assemblies,
  it does not perform de novo assembly
- The reference genomes (not contigs) are the alignment target — use Nucmer-based
  contig-to-reference alignment for reference genome coverage instead
- You only need assembly length metrics (total size, N50/N90), which the article notes
  are 'not very useful for comparing metagenome assembly performance'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) whose read-mapping coverage
    is being validated
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: BWA
  role: Indexes contig FASTA references and maps trimmed reads to the final MeGAMerged
    contigs to generate alignments
- name: SAMtools
  role: Converts and sorts BWA SAM output to indexed BAM and parses BAM per-reference
    statistics to compute contig base coverage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:bf0f4a570e91914a04e02666420312bd0e824d9d558c21e5669a1533c9b7bd25
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metagenomic-read-mapping-to-contigs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates metagenomic assemblies (individual assembler outputs and MeGAMerged contig sets such as MergedContigs.fasta) by mapping trimmed sequencing reads back to contigs with BWA and computing per-contig/per-assembly coverage statistics with SAMtools, reproducing comparisons like the CLC bio oil-spill (~85% coverage) vs MeGAMerge-without-CLC-bio (~99% coverage) analysis.

## When to use

Use when you need to judge assembly quality by read support: you have assembled contig FASTA files (individual or merged) and matching trimmed reads, and want per-contig base-coverage percentages.

## When NOT to use

- Input is raw reads without assembled contigs — this skill validates assemblies, it does not perform de novo assembly
- The reference genomes (not contigs) are the alignment target — use Nucmer-based contig-to-reference alignment for reference genome coverage instead
- You only need assembly length metrics (total size, N50/N90), which the article notes are 'not very useful for comparing metagenome assembly performance'

## Inputs

- Final MeGAMerged contig FASTA (MergedContigs.fasta)
- Individual assembler contig FASTA files (e.g., CLC bio, SOAPdenovo, Velvet, Newbler, IDBA, Ray assemblies)
- Trimmed sample reads (e.g., Illumina HiSeq 2000 shotgun reads from HMP, MetaHit, or Horizon oil spill samples)

## Outputs

- SAM/BAM alignment files plus BAM index files per contig reference
- Per-contig and per-assembly coverage statistics (percent of contig bases covered, excluding and including multiple alignments per read)
- Aggregated CSV table (contig_coverage.csv) reproducing Figure 4 and Supplementary Tables S5-S6

## How to apply

Index each contig FASTA — the final MergedContigs.fasta from MeGAMerge plus each individual assembler's contig FASTA — as a BWA reference database. Map the trimmed sample reads (trimmed with the sliding-window protocol, minimum quality score 2) to each reference with BWA to produce SAM output, then convert and sort the alignments with SAMtools into BAM files with indexes. Parse each BAM with SAMtools per-reference statistics (flagstat/idxstats-style) to compute the percentage of contig bases covered by mapped reads, both excluding and including multiple alignments per read. Aggregate per-contig and per-assembly values into a single table (contig_coverage.csv) so assembler-specific coverage deficits, such as CLC bio assemblies at ~85% (88.2% counting multiple alignments) versus ~99% for MeGAMerge contigs assembled without CLC bio, can be compared.

## Related tools

- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) whose read-mapping coverage is being validated) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **BWA** (Indexes contig FASTA references and maps trimmed reads to the final MeGAMerged contigs to generate alignments)
- **SAMtools** (Converts and sorts BWA SAM output to indexed BAM and parses BAM per-reference statistics to compute contig base coverage)

## Examples

```
bwa index MergedContigs.fasta && bwa mem MergedContigs.fasta reads.trimmed.fastq | samtools sort -o sorted.bam && samtools index sorted.bam
```

## Evaluation signals

- CLC bio oil-spill sample contig coverage falls in a plausible range around the reported ~85% (88.2% when counting multiple alignments per read)
- MeGAMerge contigs assembled without CLC bio reach near the reported ~99% average coverage
- Sorted BAM files are accompanied by valid index files (e.g., .bai) for every contig reference
- contig_coverage.csv contains per-contig and per-assembly rows for both multiple-alignment-inclusive and exclusive coverage values
- Individual CLC bio oil-spill assemblies show coverage well below the >= 99% expected for well-supported assemblies

## Limitations

- Multiple alignments per read inflate coverage estimates (e.g., 88.2% vs ~85% for the CLC bio oil-spill contigs), so both counting modes should be reported
- Total assembly size and N50/N90 comparisons are confounded because MeGAMerge removes very small contigs (<2 kb) before merging
- The merging process can introduce new chimeras, which read-mapping coverage alone will not flag

## Evidence
