---
name: assembly-validation-by-read-mapping
description: Use when assembled metagenomic contigs (e.g., MergedContigs.fasta or
  individual assembler FASTA files) plus the original trimmed reads are available
  and you need read-mapping evidence of contig coverage quality, e.g., to compare
  assemblers like CLC bio vs. MeGAMerge.
when_to_use_negative:
- Input is only raw reads with no assembled contigs — use a de novo assembler (e.g.,
  SOAPdenovo, CLC Bio, Velvet) instead of mapping back to existing contigs.
- The goal is to quantify reference genome coverage by aligning contigs to reference
  genomes — use Nucmer alignment rather than read mapping.
- Assemblies consist solely of short contigs (<2 kb) that would be removed before
  Prodigal gene prediction, making coverage statistics unrepresentative of the final
  contig set.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MeGAMerge
  role: Produces the merged MergedContigs.fasta whose coverage is validated by read
    mapping
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: BWA
  role: Indexes contig FASTA references and maps trimmed reads to produce SAM alignments
- name: SAMtools
  role: Converts SAM to sorted BAM, builds indexes, and parses BAMs for per-reference
    coverage statistics
- name: CLC Bio de novo assembler
  role: Source of individual assembly contigs compared against MeGAMerged contigs
    via read-mapping coverage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:a6ad56c084357a9a53d5133ff302da653152077898541f2a77103a9708bf92a5
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
  - CLC Bio de novo assembler
  - MeGAMerge
  - SAMtools
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# assembly-validation-by-read-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate metagenome assemblies (individual assembler outputs and MeGAMerged contigs) by mapping trimmed reads back to contigs with BWA/SAMtools and computing per-contig and per-assembly coverage percentages, exposing assembler-specific coverage deficiencies such as CLC bio contigs in the Horizon oil spill sample.

## When to use

Use when assembled metagenomic contigs (e.g., MergedContigs.fasta or individual assembler FASTA files) plus the original trimmed reads are available and you need read-mapping evidence of contig coverage quality, e.g., to compare assemblers like CLC bio vs. MeGAMerge.

## When NOT to use

- Input is only raw reads with no assembled contigs — use a de novo assembler (e.g., SOAPdenovo, CLC Bio, Velvet) instead of mapping back to existing contigs.
- The goal is to quantify reference genome coverage by aligning contigs to reference genomes — use Nucmer alignment rather than read mapping.
- Assemblies consist solely of short contigs (<2 kb) that would be removed before Prodigal gene prediction, making coverage statistics unrepresentative of the final contig set.

## Inputs

- Final MeGAMerged contig FASTA (MergedContigs.fasta)
- Individual assembler contig FASTA files (e.g., CLC bio, SOAPdenovo, Velvet, IDBA, Ray assemblies)
- Trimmed metagenomic sample reads (e.g., Horizon oil spill Illumina reads)

## Outputs

- Sorted, indexed BAM files per contig reference
- Per-contig and per-assembly coverage percentages (excluding and including multiple alignments per read)
- contig_coverage.csv aggregating coverage values (reproducing Figure 4 and Supplementary Tables S5-S6)

## How to apply

Index each contig FASTA (the final MeGAMerged MergedContigs.fasta and each individual assembly's contigs) as a BWA reference, then map the trimmed sample reads with BWA to produce SAM output. Convert and sort alignments with SAMtools into BAM files and build indexes. Parse each BAM with SAMtools per-reference statistics (flagstat/idxstats-style) to compute the percentage of contig bases covered by mapped reads, both excluding and including multiple alignments per read. Aggregate per-contig and per-assembly coverage into a single CSV table (contig_coverage.csv) reproducing Figure 4 and Supplementary Tables S5-S6; expect the CLC bio oil-spill assembly around ~85% (88.2% counting multiple alignments per read) versus ~99% for MeGAMerge contigs assembled without CLC bio, which questions individual-assembler suitability for complex samples.

## Related tools

- **MeGAMerge** (Produces the merged MergedContigs.fasta whose coverage is validated by read mapping) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **BWA** (Indexes contig FASTA references and maps trimmed reads to produce SAM alignments)
- **SAMtools** (Converts SAM to sorted BAM, builds indexes, and parses BAMs for per-reference coverage statistics)
- **CLC Bio de novo assembler** (Source of individual assembly contigs compared against MeGAMerged contigs via read-mapping coverage)

## Examples

```
perl MeGAMerge-1.0.pl outdir assembly1.fasta assembly2.fasta; bwa index MergedContigs.fasta; bwa mem MergedContigs.fasta reads.trimmed.fastq > aln.sam; samtools sort aln.sam -o aln.bam; samtools index aln.bam; samtools flagstat aln.bam
```

## Evaluation signals

- CLC bio oil-spill contig coverage falls in a plausible range around the reported ~85% (88.2% when counting multiple alignments per read).
- MeGAMerge contigs assembled without CLC bio show coverage near the reported ~99%.
- Every contig FASTA has a corresponding sorted, indexed BAM file with no unmapped-reference errors in SAMtools output.
- contig_coverage.csv contains both coverage variants (excluding and including multiple alignments per read) for each assembly.
- Per-assembly coverage aggregates reproduce the ordering in Figure 4: MeGAMerge coverage ≥ individual CLC bio coverage for the Horizon oil spill sample.

## Limitations

- Multiple alignments per read inflate coverage estimates (e.g., 85% vs 88.2% for the CLC bio oil-spill assembly), so both accounting modes must be reported.
- Removal of contigs smaller than 2 kb prior to analysis impacts total assembly size and N50/N90 metrics, which are themselves described as not very useful for comparing metagenome assembly performance.
- Overlap-based consensus tools used upstream (Minimus2/Newbler within MeGAMerge) have run-times that increase exponentially with input dataset size, limiting scalability on very large contig pools.

## Evidence

- [discussion] Additional statistics are gathered by mapping reads to final MeGAMerged contigs using BWA19. The resulting BAM file was parsed using SAMtools: "Additional statistics are gathered by mapping reads to final MeGAMerged contigs using BWA19. The resulting BAM file was parsed using SAMtools"
