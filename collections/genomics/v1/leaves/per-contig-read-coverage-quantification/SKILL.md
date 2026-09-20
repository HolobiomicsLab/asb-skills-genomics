---
name: per-contig-read-coverage-quantification
description: Use when validating metagenome assemblies by measuring what fraction
  of contig bases are covered by mapped reads — e.g. to detect poorly covered assembler
  outputs like the CLC bio Horizon oil-spill assembly (~85% coverage vs ~99% for MeGAMerge).
when_to_use_negative:
- Input is raw unassembled reads with no contig FASTA to map against — coverage validation
  presupposes existing assemblies
- Reads are untrimmed low-quality data — the article's coverage statistics assume
  reads trimmed with a sliding window protocol at minimum quality score 2
- The goal is reference-genome coverage assessment rather than read-based contig coverage
  — that requires aligning contigs to genomes with Nucmer instead
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BWA
  role: Indexes contig FASTA references and maps trimmed sample reads to contigs producing
    SAM alignments
- name: SAMtools
  role: Converts, sorts, indexes SAM/BAM alignments and parses BAM per-reference statistics
    to compute contig base coverage
- name: MeGAMerge
  role: Produces the MergedContigs.fasta whose coverage is compared against individual
    assembler contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:8e98341cb132bcddb6d36645beeb92a7cf2789bcf1bda48bac80acd186faf052
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
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Per-contig read coverage quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the percentage of bases on each assembled contig (individual assembler output or MeGAMerge MergedContigs.fasta) covered by trimmed sample reads, using BWA mapping parsed with SAMtools, to validate assembly quality and expose assembler-specific coverage deficits.

## When to use

Use when validating metagenome assemblies by measuring what fraction of contig bases are covered by mapped reads — e.g. to detect poorly covered assembler outputs like the CLC bio Horizon oil-spill assembly (~85% coverage vs ~99% for MeGAMerge).

## When NOT to use

- Input is raw unassembled reads with no contig FASTA to map against — coverage validation presupposes existing assemblies
- Reads are untrimmed low-quality data — the article's coverage statistics assume reads trimmed with a sliding window protocol at minimum quality score 2
- The goal is reference-genome coverage assessment rather than read-based contig coverage — that requires aligning contigs to genomes with Nucmer instead

## Inputs

- Contig FASTA files from individual assemblies (e.g. CLC bio, Newbler, SOAPdenovo, Velvet, IDBA, Ray) and the MeGAMerge final MergedContigs.fasta
- Quality-trimmed sample reads (e.g. Horizon oil spill Illumina data) in FASTQ format

## Outputs

- BAM files with indexes for each contig reference
- Per-contig and per-assembly percentage-of-bases-covered values, aggregated into contig_coverage.csv (reproducing Figure 4 and Supplementary Tables S5-S6)

## How to apply

Index each contig FASTA (the final MergedContigs.fasta from MeGAMerge and each individual assembly's contigs) as a BWA reference database, then map the quality-trimmed sample reads (trimmed with the sliding-window protocol, minimum quality score 2) with BWA to produce SAM output. Convert and sort alignments with SAMtools into BAM files with indexes. Parse each BAM with SAMtools per-reference statistics to compute the percentage of contig bases covered by mapped reads, both excluding and including multiple alignments per read. Aggregate per-contig and per-assembly values into a single CSV (contig_coverage.csv) for comparison across assemblers; low average coverage of an individual assembly (e.g. CLC bio on the Horizon oil spill sample) indicates that assembler may be unsuitable for complex metagenome samples.

## Related tools

- **BWA** (Indexes contig FASTA references and maps trimmed sample reads to contigs producing SAM alignments)
- **SAMtools** (Converts, sorts, indexes SAM/BAM alignments and parses BAM per-reference statistics to compute contig base coverage)
- **MeGAMerge** (Produces the MergedContigs.fasta whose coverage is compared against individual assembler contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
bwa index MergedContigs.fasta; bwa mem MergedContigs.fasta trimmed_R1.fastq trimmed_R2.fastq | samtools sort -o aln.bam; samtools index aln.bam; samtools idxstats aln.bam
```

## Evaluation signals

- CLC bio Horizon oil-spill sample contig coverage falls near the reported ~85% (88.2% when counting multiple alignments per read), well below the expected >=99%
- MeGAMerge contigs assembled without CLC bio show ~99% read coverage
- Every contig FASTA has a successfully generated BWA index and every BAM has a companion index file before statistics are computed
- Coverage is reported both excluding and including multiple alignments per read, so the two per-read-counting modes can be cross-checked
- Aggregated CSV reproduces the per-contig and per-assembly structure of Figure 4 and Supplementary Tables S5-S6

## Limitations

- Coverage percentages depend on read trimming quality; untrimmed or differently trimmed reads will shift coverage values away from the published benchmarks
- Counting multiple alignments per read changes results substantially (85% vs 88.2% for the CLC bio oil-spill sample), so the counting mode must be stated when comparing coverage
- Very small contigs (<2 kb) are removed from MeGAMerge inputs, so coverage statistics describe the >=2 kb contig population only

## Evidence

- [results] Finding on CLC bio coverage deficit: [evidence span withheld — anchor status: not_found]
