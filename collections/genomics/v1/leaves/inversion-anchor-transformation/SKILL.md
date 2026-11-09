---
name: inversion-anchor-transformation
description: Use when mapping long noisy PacBio or Oxford Nanopore reads to a reference
  genome and inversion structural variants must be detected and reported as candidate
  SV calls in SAM format.
when_to_use_negative:
- Reads are short rather than long noisy reads — short reads less frequently span
  the breakpoints of structural variations, which is the regime invMap targets
- The machine has less than 16G of RAM when mapping sequence datasets to large reference
  genomes
- The analysis goal is generic read alignment with no interest in inversion structural
  variants
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: 'Primary tool: chains seed anchors with a dynamic-programming scoring method,
    transforms nonlinear anchors to linear form to detect inversions, and fills gaps
    with KSW2 before emitting SAM output'
  repo: https://github.com/zhang134/invMap.git
- name: KSW2
  role: Gap-filling alignment algorithm invoked by invMap to complete the chained
    alignment in remaining gaps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:afc439b9bdbd2810f91fa3b3b26a81585f1951c28adf73ed96cf1ea7bfe06996
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - KSW2
  - invMap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# inversion-anchor-transformation

## Summary

Reconstructs the invMap four-step long-read mapping pipeline (minimizer indexing, chaining-based aligned-region location, nonlinear-to-linear anchor transformation for inversion detection, gap-filling with KSW2) to detect inversion structural variants in long noisy PacBio/Oxford Nanopore reads.

## When to use

Use when mapping long noisy PacBio or Oxford Nanopore reads to a reference genome and inversion structural variants must be detected and reported as candidate SV calls in SAM format.

## When NOT to use

- Reads are short rather than long noisy reads — short reads less frequently span the breakpoints of structural variations, which is the regime invMap targets
- The machine has less than 16G of RAM when mapping sequence datasets to large reference genomes
- The analysis goal is generic read alignment with no interest in inversion structural variants

## Inputs

- Reference genome in FASTA format (genome.fa)
- Long noisy sequencing reads in FASTA or FASTQ format (e.g. PacBio or Oxford Nanopore)

## Outputs

- invMap_aligned.sam — final mapped alignments in SAM format, with inversions reported as candidate SV calls

## How to apply

First build a minimizer hash-table index of the reference genome (genome.fa) so seed anchors for each long noisy read can be retrieved rapidly. For each read, apply invMap's specifically designed dynamic-programming scoring method for chaining to locate the aligned region. Then examine the remaining anchors inside the located aligned region and transform nonlinear anchors to linear form (with boundary region extension) so non-co-linear anchors become co-linear, exposing potential inversion chains — this is the key step that enables discovery of inversions even of small size. Finally fill remaining gaps in the chained alignment with the KSW2 alignment algorithm and write results to invMap_aligned.sam (e.g. ./invMap genome.fa reads.fa > invMap_aligned.sam). Compile the code with make and check options via ./invmap; validate that the SAM output is parseable and its inversion records correspond to candidate SV calls.

## Related tools

- **invMap** (Primary tool: chains seed anchors with a dynamic-programming scoring method, transforms nonlinear anchors to linear form to detect inversions, and fills gaps with KSW2 before emitting SAM output) — https://github.com/zhang134/invMap.git
- **KSW2** (Gap-filling alignment algorithm invoked by invMap to complete the chained alignment in remaining gaps)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap_aligned.sam exists and is parseable SAM with aligned records
- Inversion events in the SAM output are reported as candidate SV calls, including inversions of small size
- On simulated datasets, aligned-region location and SV calling accuracy for inversions is at least as good as competing methods across different genomes and sequencing coverages
- On the NA12878 real human genome sequencing dataset, invMap finds more candidate inversion variant calls than competing methods
- The two-step setup compiles with make and ./invmap prints help information and options explanation

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Requires approximately 800 M of available disk space on the drive or partition
- Boundary region extension during nonlinear-to-linear anchor transformation is needed, implying anchors near inversion breakpoints may need careful handling
- No changelog found for the zhang134/invMap repository

## Evidence

- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [intro] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Input file requires fasta or fastq format: "Input file requires fasta or fastq format"
