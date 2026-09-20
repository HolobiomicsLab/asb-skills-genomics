---
name: read-coverage-correlation-analysis
description: Use when comparing scaffolding graphs built from the same Illumina read
  set by different aligners (Bowtie, Bowtie2, Bwa, Bwa-Mem, Fast-SG) and you need
  a quantitative, per-genome measure of contig coverage agreement.
when_to_use_negative:
- Long-read scaffolding evaluation (PacBio/Nanopore) where the comparison target is
  Links-style hybrid scaffolding rather than short-read aligner coverage agreement
- When only final scaffold sequences are available without the underlying scaffolding
  graphs, since per-contig read-coverage cannot be computed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Alignment-free method whose scaffolding-graph contig read-coverage values
    serve as the reference for pairwise Pearson correlation against aligner-built
    graphs
  repo: https://github.com/adigenova/fast-sg
- name: Bowtie2
  role: Short-read aligner used to build comparison scaffolding graphs from the same
    Illumina read sets
- name: Bwa-Mem
  role: Short-read aligner used to build comparison scaffolding graphs from the same
    Illumina read sets
- name: PERL
  role: Wrapper language for FAST-SG.pl used to generate the Fast-SG graphs across
    k-mer sizes in [12-256]
- name: C++
  role: Implementation language of Fast-SG's ultrafast alignment-free graph construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:e06b5c29575c6509020fcb7f68715247da4e4f8dbc8309fd25e1fcfb3dc3c5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Bowtie2
  - Bwa-Mem
  - C++
  - Fast-SG
  - PERL
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# read-coverage-correlation-analysis

## Summary

Computes average contig read-coverage for scaffolding graphs built by short-read aligners and derives pairwise Pearson correlations against Fast-SG's coverage values, to quantify agreement between alignment-free and alignment-based graph construction.

## When to use

Use when comparing scaffolding graphs built from the same Illumina read set by different aligners (Bowtie, Bowtie2, Bwa, Bwa-Mem, Fast-SG) and you need a quantitative, per-genome measure of contig coverage agreement.

## When NOT to use

- Long-read scaffolding evaluation (PacBio/Nanopore) where the comparison target is Links-style hybrid scaffolding rather than short-read aligner coverage agreement
- When only final scaffold sequences are available without the underlying scaffolding graphs, since per-contig read-coverage cannot be computed

## Inputs

- Contig FASTA file for each genome test case
- Illumina read set / read configuration file (e.g. examples/ecoli-reads.txt style library file)
- Scaffolding graphs built by each short-read aligner (Bowtie, Bowtie2, Bwa, Bwa-Mem)
- Fast-SG synthetic libraries / scaffolding graph built with FAST-SG.pl

## Outputs

- Per-contig average read-coverage values for each aligner-built graph and the Fast-SG graph
- Pairwise Pearson correlation coefficients comparing each aligner's coverage against Fast-SG's
- Benchmark results table with one row per genome-scaffolder combination, including coverage correlation columns

## How to apply

For each genome test case (e.g. S. aureus, R. sphaeroides, P. falciparum, human chr14), take the scaffolding graphs produced by each short-read aligner and compute the average read-coverage per contig in the graph. Do the same for the Fast-SG graph built with FAST-SG.pl over the supported k-mer range [12-256]. Then derive pairwise Pearson correlation coefficients between each aligner's contig read-coverage values and Fast-SG's values, recording a numeric coefficient for every aligner comparison. These correlations complement F-score/error tables compiled per genome-scaffolder combination (Opera-LG, Besst2, ScaffMatch, Boss) and support the finding that Fast-SG outperforms state-of-the-art short-read aligners when building the scaffolding graph.

## Related tools

- **Fast-SG** (Alignment-free method whose scaffolding-graph contig read-coverage values serve as the reference for pairwise Pearson correlation against aligner-built graphs) — https://github.com/adigenova/fast-sg
- **Bowtie2** (Short-read aligner used to build comparison scaffolding graphs from the same Illumina read sets)
- **Bwa-Mem** (Short-read aligner used to build comparison scaffolding graphs from the same Illumina read sets)
- **PERL** (Wrapper language for FAST-SG.pl used to generate the Fast-SG graphs across k-mer sizes in [12-256])
- **C++** (Implementation language of Fast-SG's ultrafast alignment-free graph construction)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- The benchmark results table contains a numeric Pearson correlation coefficient for each aligner (Bowtie, Bowtie2, Bwa, Bwa-Mem) comparison against Fast-SG
- The F-score/error table contains rows for all four genomes x five graph-construction methods (Fast-SG plus four aligners)
- Per-contig coverage values are computable for every contig node in each aligner-built graph, with no missing entries
- Coverage correlations are consistent with the F-score ranking: Fast-SG consistently reaches better scaffolding results than the short-read aligners across the four test cases

## Limitations

- The synthetic library insert size is restricted to the actual long-read size, which bounds the linking information available to the graph
- Gap size estimation with long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is challenging because linking information comes from a range of insert sizes and read-pair orientation may be unknown, complicating coverage-based comparisons in those settings
- Results are specific to the four real Illumina test cases (S. aureus, R. sphaeroides, P. falciparum, human chr14) and the scaffolders benchmarked (Opera-LG, Besst2, ScaffMatch, Boss)

## Evidence

- [other] Compute the average contig read-coverage for each aligner-built graph and derive pairwise Pearson correlations against Fast-SG's values: [evidence span withheld — anchor status: not_found]
- [abstract] Fast-SG outperforms state-of-the-art short-read aligners: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [methods] run Fast-SG wrapper with k-mer size: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] Fast-SG is faster than current state-of-the-art short-read aligners: "Fast-SG is faster than the current state-of-the-art short-read aligners"
- [readme] supported k-mer range: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
