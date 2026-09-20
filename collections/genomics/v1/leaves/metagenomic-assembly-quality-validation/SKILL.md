---
name: metagenomic-assembly-quality-validation
description: Use when a merged or pooled metagenomic contig set (e.g. from MeGAMerge)
  must be quality-checked against raw reads and known reference genomes to confirm
  contig reduction, coverage maintenance, chimera counts, and removal of unmapped
  contigs.
when_to_use_negative:
- You have only a single-assembler contig set with no reference genomes available
  — reference coverage and chimera validation cannot be computed
- You are still choosing between assemblers rather than validating a merged/pooled
  contig set — the article instead recommends comparing assembly metrics via read
  mapping and gene prediction per assembly
- Your goal is to judge assembly quality by total assembly size or N50/N90 alone —
  the paper states these metrics 'are not very useful for comparing metagenome assembly
  performance'
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: BWA
  role: Maps reads back to the final MeGAMerged contigs to gather mapping statistics
- name: SAMtools
  role: Parses the BWA BAM file to count unmapped reads/contigs
- name: Nucmer (MUMmer)
  role: Aligns contigs to reference genomes to compute reference genome coverage
- name: Prodigal
  role: Gene prediction support on contigs ≥2 kb; contigs smaller than 2 kb removed
    before prediction
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) that this validation
    assesses
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:b0200bb85228ec268b80fdae37d556adee25dfd052d3d7ce638be8cbea346a98
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
  - MeGAMerge
  - MetaSim
  - Minimus2
  - Nucmer (MUMmer)
  - Prodigal
  - Ray
  - SAMtools
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# metagenomic-assembly-quality-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a merged (MeGAMerged) metagenomic contig set by mapping reads back with BWA/SAMtools, aligning contigs to references with Nucmer for coverage, and flagging chimeras — confirming that merging reduces contig count and unmapped contigs while maintaining reference genome coverage.

## When to use

Use when a merged or pooled metagenomic contig set (e.g. from MeGAMerge) must be quality-checked against raw reads and known reference genomes to confirm contig reduction, coverage maintenance, chimera counts, and removal of unmapped contigs.

## When NOT to use

- You have only a single-assembler contig set with no reference genomes available — reference coverage and chimera validation cannot be computed
- You are still choosing between assemblers rather than validating a merged/pooled contig set — the article instead recommends comparing assembly metrics via read mapping and gene prediction per assembly
- Your goal is to judge assembly quality by total assembly size or N50/N90 alone — the paper states these metrics 'are not very useful for comparing metagenome assembly performance'

## Inputs

- Final merged contig set (e.g. MergedContigs.fasta from MeGAMerge)
- Dereplicated pooled input contigs (pre-merge, for before/after comparison)
- Original FASTQ/FASTA read set (e.g. simulated MetaSim shotgun reads)
- Reference genome sequences (e.g. the 1,000 reference genomes)
- BAM file from BWA read mapping

## Outputs

- Unmapped contig/read counts parsed from SAMtools on the BWA BAM
- Reference genome coverage per genome from Nucmer alignment (e.g. validation_summary.csv with 1,000 per-genome rows)
- Chimera counts before merging, after merging, and carried over from dereplicated inputs
- Contig count reduction totals (input vs merged, e.g. 11.7M → <3.7M)

## How to apply

Map the original reads back to the final contigs with BWA, parse the resulting BAM file with SAMtools to count unmapped reads/contigs. Align contigs to reference genomes with Nucmer to obtain reference genome coverage. Flag contigs that did not contiguously align to a single reference genome as chimeric, and tally chimeras before merging, after merging, and carried over from dereplicated inputs. On the synthetic 1,000-genome benchmark, the correct outcome is ~11.7M dereplicated input contigs (80.26% coverage, 318 unmapped) reduced to under 3.7M contigs with 80.25% coverage and only 114 unmapped contigs, with chimeras dropping from 10,087 to 9,225 (6,940 carried over). Additional support can come from gene prediction (Prodigal on contigs ≥2 kb) and similarity searches.

## Related tools

- **BWA** (Maps reads back to the final MeGAMerged contigs to gather mapping statistics)
- **SAMtools** (Parses the BWA BAM file to count unmapped reads/contigs)
- **Nucmer (MUMmer)** (Aligns contigs to reference genomes to compute reference genome coverage)
- **Prodigal** (Gene prediction support on contigs ≥2 kb; contigs smaller than 2 kb removed before prediction)
- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) that this validation assesses) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
bwa mem MergedContigs.fasta reads.fastq | samtools view -c -f 4 -; nucmer --prefix=refcov refs/ refcontigs/ && delta-filter -r -q refcov.delta > refcov.filt && show-coords -r refcov.filt
```

## Evaluation signals

- Contig count drops substantially after merging (benchmark: 11.7M dereplicated input contigs → <3.7M merged contigs)
- Unmapped contigs decrease after merging (benchmark: 318 unmapped inputs vs 114 unmapped merged contigs)
- Reference genome coverage is maintained (benchmark: 80.26% pre-merge vs 80.25% post-merge)
- Total chimeras decrease even though merging introduces new ones (benchmark: 10,087 → 9,225, with 6,940 carried over)
- validation_summary.csv contains 1,000 per-genome coverage rows (row_count_equals check) with totals within expected ranges

## Limitations

- Merging introduces new chimeras even though total chimeras decrease
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics
- Total assembly size and N50/N90 are not very useful for comparing metagenome assembly performance
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- For single-genome inputs, use -single_genome=1 so Newbler runs once and fewer contigs are excluded

## Evidence
