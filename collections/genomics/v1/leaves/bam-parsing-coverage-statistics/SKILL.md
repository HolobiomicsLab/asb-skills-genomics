---
name: bam-parsing-coverage-statistics
description: Use when validating metagenome contig assemblies (e.g., MeGAMerge MergedContigs.fasta
  or individual CLC bio/Velvet/SOAPdenovo assemblies) by quantifying what fraction
  of contig bases are covered by mapped reads.
when_to_use_negative:
- No contig FASTA exists yet — coverage validation requires assembled contigs to serve
  as BWA reference databases, not raw unassembled reads
- Estimating reference genome coverage or identifying chimeric contigs — the article
  obtains reference genome coverage by aligning contigs to genome sequences using
  Nucmer, not by read mapping
- Comparing assemblies on total assembly size or N50/N90 — the article notes these
  are 'not very useful for comparing metagenome assembly performance', and this skill
  yields coverage percentages instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0193
- http://edamontology.org/topic_0622
tools:
- name: BWA
  role: Maps trimmed sample reads against each contig FASTA reference (MergedContigs.fasta
    and individual assembly contigs) to produce SAM alignments for coverage calculation
- name: SAMtools
  role: Converts and sorts BWA SAM output into indexed BAM files and parses per-reference
    statistics (flagstat/idxstats-style) to compute percentage of contig bases covered
- name: MeGAMerge
  role: Produces the merged MergedContigs.fasta whose coverage is compared against
    individual assembler outputs to demonstrate the assembly improvement
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:03e0eecadd45f59a8da322c5bcc81ee522c0cbfb735c3f8dd7a83b255970ce7e
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0193
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bam-parsing-coverage-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates metagenome contig assemblies (the final MeGAMerged MergedContigs.fasta and individual assembler FASTAs) by mapping trimmed reads with BWA and parsing the sorted, indexed BAM with SAMtools to compute per-contig and per-assembly percentage of contig bases covered by mapped reads.

## When to use

Use when validating metagenome contig assemblies (e.g., MeGAMerge MergedContigs.fasta or individual CLC bio/Velvet/SOAPdenovo assemblies) by quantifying what fraction of contig bases are covered by mapped reads.

## When NOT to use

- No contig FASTA exists yet — coverage validation requires assembled contigs to serve as BWA reference databases, not raw unassembled reads
- Estimating reference genome coverage or identifying chimeric contigs — the article obtains reference genome coverage by aligning contigs to genome sequences using Nucmer, not by read mapping
- Comparing assemblies on total assembly size or N50/N90 — the article notes these are 'not very useful for comparing metagenome assembly performance', and this skill yields coverage percentages instead

## Inputs

- Trimmed sample reads (FASTQ)
- Final MeGAMerged contig FASTA (MergedContigs.fasta)
- Individual assembly contig FASTAs (e.g., CLC bio, Velvet, SOAPdenovo outputs, Multi-FASTA)
- BWA reference indexes built from each contig FASTA

## Outputs

- SAM files from BWA mapping of reads to each contig reference
- Sorted BAM files with associated index files
- Per-contig and per-assembly percentage of contig bases covered (excluding and including multiple alignments per read)
- contig_coverage.csv aggregating coverage values (reproducing Figure 4 and Supplementary Tables S5-S6)

## How to apply

Index the final MeGAMerged contig FASTA (MergedContigs.fasta) and each individual assembly's contig FASTA as BWA reference databases, then map the trimmed sample reads (trimmed with a sliding-window protocol at minimum quality score 2) to each reference to produce SAM output. Convert and sort alignments with SAMtools, generating BAM files with associated index files. Parse each BAM with SAMtools flagstat/idxstats-style per-reference statistics to compute the percentage of contig bases covered by mapped reads, both excluding and including multiple alignments per read. Aggregate per-contig and per-assembly coverage values into a single CSV table (contig_coverage.csv) reproducing Figure 4 and Supplementary Tables S5-S6. The rationale is that read mapping exposes assembler-specific coverage differences — e.g., the Horizon oil spill CLC bio assemblies fall to ~85% coverage (~88.2% counting multiple alignments per read) while MeGAMerge contigs assembled without CLC bio reach ~99%.

## Related tools

- **BWA** (Maps trimmed sample reads against each contig FASTA reference (MergedContigs.fasta and individual assembly contigs) to produce SAM alignments for coverage calculation)
- **SAMtools** (Converts and sorts BWA SAM output into indexed BAM files and parses per-reference statistics (flagstat/idxstats-style) to compute percentage of contig bases covered)
- **MeGAMerge** (Produces the merged MergedContigs.fasta whose coverage is compared against individual assembler outputs to demonstrate the assembly improvement) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
bwa index MergedContigs.fasta; bwa mem MergedContigs.fasta trimmed_reads.fastq > aln.sam; samtools sort -o aln.sorted.bam aln.sam; samtools index aln.sorted.bam
```

## Evaluation signals

- The CLC bio Horizon oil-spill sample coverage falls in a plausible range around the reported ~85% (~88.2% when counting multiple alignments per read)
- MeGAMerge contigs assembled without CLC bio show coverage near the reported ~99%
- Coverage is computed both excluding and including multiple alignments per read, with both values reported side by side
- All per-contig and per-assembly values aggregate into a single contig_coverage.csv matching the schema of Figure 4 and Supplementary Tables S5-S6
- Every BAM has a valid index file so SAMtools per-reference statistics (flagstat/idxstats-style) run without error

## Limitations

- Low coverage can indicate assembler-specific problems (e.g., CLC bio individual assemblies at ~85% on the oil-spill sample vs >= 99% expected), so coverage deficits must not be attributed to the mapping step alone
- The merging process introduces new chimeras, so MeGAMerged contig coverage can include chimeric sequence even while MeGAMerge reduces the total number of chimeras input
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting the scale of samples this validation pipeline can accompany

## Evidence
