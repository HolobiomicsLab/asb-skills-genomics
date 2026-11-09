---
name: minimizer-hash-table-genome-indexing
description: Use when mapping long, noisy PacBio or Oxford Nanopore reads to a large
  reference genome and you need fast seed lookup plus detection of inversion SVs via
  anchor chaining.
when_to_use_negative:
- The task is short-read mapping without structural-variant detection — invMap targets
  long noisy reads spanning SV breakpoints.
- The reference genome or read set exceeds available memory — invMap requires a minimum
  of 16G of RAM for large reference genomes.
- Input files are not in FASTA or FASTQ format and cannot be converted.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Long-read mapping tool that builds the minimizer hash-table index, chains
    seed anchors with dynamic-programming scoring, detects inversion chains, and emits
    SAM output via KSW2 gap filling
  repo: https://github.com/zhang134/invMap.git
- name: KSW2
  role: Alignment backend used inside invMap to fill remaining gaps in the chained
    alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:c5413dd598cc63de1720fd9068c984ec67e4a0b86811bc6ac1b18110a67dc13d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - KSW2
  - invMap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# minimizer-hash-table-genome-indexing

## Summary

Index a reference genome as a minimizer hash-table so that seeds for each long noisy read can be retrieved rapidly, as done by the invMap mapper to anchor PacBio/Oxford Nanopore reads and expose inversion structural variants.

## When to use

Use when mapping long, noisy PacBio or Oxford Nanopore reads to a large reference genome and you need fast seed lookup plus detection of inversion SVs via anchor chaining.

## When NOT to use

- The task is short-read mapping without structural-variant detection — invMap targets long noisy reads spanning SV breakpoints.
- The reference genome or read set exceeds available memory — invMap requires a minimum of 16G of RAM for large reference genomes.
- Input files are not in FASTA or FASTQ format and cannot be converted.

## Inputs

- Reference genome in FASTA format (e.g. genome.fa)
- Long noisy reads from PacBio or Oxford Nanopore sequencing in FASTA or FASTQ format

## Outputs

- Read-to-reference alignments with inversion chains reported, written as invMap_aligned.sam in SAM format

## How to apply

Build a minimizer hash-table index over the reference genome FASTA (e.g. genome.fa) so each read's seed anchors can be looked up quickly. For every long noisy read, retrieve anchors from the index and chain them with invMap's specifically designed dynamic-programming scoring method to locate the aligned region. Then examine remaining non-co-linear anchors inside the located region, transforming nonlinear anchors to linear form (with boundary region extension) to reveal potential inversion chains, and fill residual gaps with KSW2 gap alignment. Compile the tool once with the make command, note that 16G RAM minimum is required for large reference genomes, and inputs must be FASTA or FASTQ format.

## Related tools

- **invMap** (Long-read mapping tool that builds the minimizer hash-table index, chains seed anchors with dynamic-programming scoring, detects inversion chains, and emits SAM output via KSW2 gap filling) — https://github.com/zhang134/invMap.git
- **KSW2** (Alignment backend used inside invMap to fill remaining gaps in the chained alignment)

## Examples

```
make; ./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap_aligned.sam is produced and is parseable SAM with aligned records for the input reads.
- Inversion events, including small ones, appear as candidate SV calls derived from non-co-linear anchors that were transformed to co-linear form.
- Chaining used the specifically designed DP scoring method, i.e. aligned regions are located before inversion checks within them (per the paper's two-stage workflow).
- Benchmark against competing methods on simulated datasets shows equal or better accuracy locating aligned regions and calling inversion SVs, as reported for invMap.
- Run completes within a 16G-RAM budget on the chosen reference genome.

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.
- Minimum hardware includes roughly an Intel Core i3 2100 (Windows) and about 800 M of free disk space.
- Inversion detection depends on residual anchors inside the located aligned region, so anchors missed by chaining may hide small inversions.

## Evidence

- [abstract] For each long noisy read, invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [intro] Ram requirement for large genomes: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Running example and SAM output: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [readme] Accepted input formats: "Input file requires fasta or fastq format"
