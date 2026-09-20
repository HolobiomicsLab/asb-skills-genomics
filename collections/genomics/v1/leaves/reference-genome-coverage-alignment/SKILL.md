---
name: reference-genome-coverage-alignment
description: Use when a merged or post-assembly contig set must be validated against
  reference genomes — e.g., checking that merging maintains reference coverage while
  reducing contig count and unmapped contigs.
when_to_use_negative:
- The metagenome sample has no reference genomes available, so coverage cannot be
  computed and only ab initio metrics (gene prediction, similarity searches) apply
- The goal is de novo assembly quality assessment of a single isolate where total
  assembly size and N50/N90 suffice — the article notes these are 'not very useful
  for comparing metagenome assembly performance' but coverage-based validation targets
  mixed communities
- Input is raw reads rather than an assembled/merged contig set; this skill validates
  post-assembly output, it does not assemble
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nucmer
  role: Aligns contigs to the reference genome sequences to obtain reference genome
    coverage and identify chimeras
- name: BWA
  role: Maps reads back to the final MeGAMerged contigs to gather validation statistics
- name: SAMtools
  role: Parses the resulting BAM file from BWA to count unmapped contigs
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) that is the subject of
    the coverage validation
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: MUMmer
  role: Underlying alignment package required by Nucmer; must be compiled in 64-bit
    mode for larger genome projects
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:1253a2a7182b1b4fb64633edf2b28453b0546111178ce78a18d6ab2d840c1f6c
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
  - AMOS toolbox
  - BWA
  - IDBA
  - MUMmer
  - MeGAMerge
  - MetaSim
  - Minimus2
  - Nucmer
  - Ray
  - SAMtools
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# reference-genome-coverage-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate a merged metagenome assembly (e.g., MeGAMerged contigs) by aligning contigs to known reference genomes with Nucmer to compute reference genome coverage, and by mapping reads back with BWA/SAMtools to count unmapped contigs and flag chimeras.

## When to use

Use when a merged or post-assembly contig set must be validated against reference genomes — e.g., checking that merging maintains reference coverage while reducing contig count and unmapped contigs.

## When NOT to use

- The metagenome sample has no reference genomes available, so coverage cannot be computed and only ab initio metrics (gene prediction, similarity searches) apply
- The goal is de novo assembly quality assessment of a single isolate where total assembly size and N50/N90 suffice — the article notes these are 'not very useful for comparing metagenome assembly performance' but coverage-based validation targets mixed communities
- Input is raw reads rather than an assembled/merged contig set; this skill validates post-assembly output, it does not assemble

## Inputs

- Final merged contig set in Multi-FASTA format (e.g., MergedContigs.fasta from MeGAMerge)
- Reference genome sequences for the simulated or known community (1,000 genomes, Supplementary Table S8)
- Original trimmed shotgun reads (e.g., MetaSim-simulated Illumina reads, one HiSeq 2000 lane equivalent) in FASTA/FASTQ
- BAM file from BWA read mapping, parsed with SAMtools

## Outputs

- Per-reference-genome coverage values (validation_summary.csv with 1,000 per-genome coverage rows)
- Count of unmapped contigs before and after merging (318 vs 114)
- Total and per-genome reference genome coverage percentages (80.26% vs 80.25%)
- Chimera tallies: before merging, after merging, and carried over from dereplicated inputs (10,087 vs 9,225, 6,940 carried over)

## How to apply

Take the final merged Multi-FASTA contig set (default MeGAMerge output MergedContigs.fasta) produced by the MeGAMerge pipeline (dereplication, 2.0 kb binning, Newbler reassembly of small contigs, Minimus2 merging). Align the contigs to the reference genome sequences using Nucmer to obtain per-reference genome coverage. Separately, map the original reads back to the final contigs using BWA and parse the resulting BAM with SAMtools to count unmapped contigs; flag as chimeric any contig that did not contiguously align to a single reference genome, tallying chimeras before merging, after merging, and carried over from the dereplicated inputs. Judge success by the article's synthetic 1,000-genome benchmark: contig count drops from 11.7M dereplicated inputs to under 3.7M, unmapped contigs drop from 318 to 114, reference coverage is maintained (80.26% to 80.25%), and chimeras drop from 10,087 to 9,225 with 6,940 carried over.

## Related tools

- **Nucmer** (Aligns contigs to the reference genome sequences to obtain reference genome coverage and identify chimeras)
- **BWA** (Maps reads back to the final MeGAMerged contigs to gather validation statistics)
- **SAMtools** (Parses the resulting BAM file from BWA to count unmapped contigs)
- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) that is the subject of the coverage validation) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **MUMmer** (Underlying alignment package required by Nucmer; must be compiled in 64-bit mode for larger genome projects)

## Examples

```
perl MeGAMerge-1.0.pl -o=MergedContigs.fasta output_dir contigs1.fasta contigs2.fasta; nucmer refgenomes/ merged/MergedContigs.fasta; bwa mem merged/MergedContigs.fasta reads.fq > aln.sam
```

## Evaluation signals

- Merged contig count falls below 3.7M from 11.7M dereplicated input contigs (synthetic 1,000-genome benchmark)
- Reference genome coverage is essentially unchanged: 80.26% for dereplicated inputs vs 80.25% after merging
- Unmapped contig count decreases from 318 (inputs) to 114 (merged output)
- Chimera total drops from 10,087 to 9,225 with 6,940 carried over, confirming merging reduces net chimeras despite introducing new ones
- validation_summary.csv contains exactly one coverage row per reference genome (row_count_equals 1,000) with totals within expected range

## Limitations

- The merging process does introduce new chimeras, even though the total number of input chimeras is reduced
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- Coverage-based validation requires known reference genomes; it cannot be applied to entirely novel communities
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, so those metrics are poor comparators here

## Evidence
