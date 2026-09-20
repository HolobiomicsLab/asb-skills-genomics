---
name: genome-assembly-fscore-evaluation
description: Use when comparing scaffolding-graph construction methods or scaffolders
  on real read data and you need a reference-based F-score/error table plus coverage-correlation
  diagnostics to decide which builder wins.
when_to_use_negative:
- No reference genome is available for the assembled organism, since F-score and error
  counts are computed relative to a reference
- The question is scaffolding-graph construction itself (running FAST-SG.pl or an
  aligner) rather than benchmarking/scoring the resulting scaffolds
- Evaluating gap-size estimation with long-range mate-pair technologies (10X Genomics,
  DOVETAIL), where linking information spans a range of insert sizes and read-pair
  orientation may be unknown, complicating error attribution
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Fast-SG
  role: Alignment-free construction of the scaffolding graph (k-mer sizes 12-256 via
    FAST-SG.pl) whose scaffold outputs set the benchmark baseline against aligner-built
    graphs
  repo: https://github.com/adigenova/fast-sg
- name: PERL
  role: Wrapping the Fast-SG pipeline (FAST-SG.pl) used to generate the graphs being
    benchmarked
  repo: https://github.com/adigenova/fast-sg
- name: C++
  role: Implementation language of Fast-SG whose graph-construction performance is
    scored in the F-score/error benchmark
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:7b077b02af6bdf5f8a0907aed1b579b4e9a748a31cec477887c4edbe14ffe2d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - C++
  - Fast-SG
  - PERL
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# genome-assembly-fscore-evaluation

## Summary

Benchmarks scaffolding graphs and scaffold outputs by computing F-score and error counts against a reference genome, then aggregating per-genome results (e.g., four genomes x five graph-construction methods) into tables plus Pearson correlations of contig read-coverage. This is how Fast-SG demonstrated that its alignment-free scaffolding graph beats Bowtie, Bowtie2, BWA, and BWA-MEM-built graphs.

## When to use

Use when comparing scaffolding-graph construction methods or scaffolders on real read data and you need a reference-based F-score/error table plus coverage-correlation diagnostics to decide which builder wins.

## When NOT to use

- No reference genome is available for the assembled organism, since F-score and error counts are computed relative to a reference
- The question is scaffolding-graph construction itself (running FAST-SG.pl or an aligner) rather than benchmarking/scoring the resulting scaffolds
- Evaluating gap-size estimation with long-range mate-pair technologies (10X Genomics, DOVETAIL), where linking information spans a range of insert sizes and read-pair orientation may be unknown, complicating error attribution

## Inputs

- Contig FASTA assemblies for each test genome (S. aureus, R. sphaeroides, P. falciparum, human chr14)
- Real Illumina read sets plus a read configuration file describing the library (e.g., examples/ecoli-reads.txt format)
- Scaffolding graphs built by each method (Fast-SG synthetic libraries vs. Bowtie/Bowtie2/BWA/BWA-MEM alignments)
- Scaffold outputs from scaffolders Opera-LG, Besst2, ScaffMatch, and Boss
- Reference genome sequences for each test case

## Outputs

- Per-genome F-score and error-count tables for each genome x scaffolder combination
- Average contig read-coverage values per aligner-built graph
- Pairwise Pearson correlation coefficients of read-coverage (each aligner vs. Fast-SG)
- A single benchmark results file with one row per genome-scaffolder combination (four genomes x five graph-construction methods)

## How to apply

For each test genome (here S. aureus, R. sphaeroides, P. falciparum, human chr14), build scaffolding graphs from the same Illumina read sets with each competing method: the alignment-free Fast-SG wrapper FAST-SG.pl with k-mer size in the supported range [12-256], and short-read aligners Bowtie, Bowtie2, BWA, BWA-MEM. Feed each graph to the short-read scaffolders Opera-LG, Besst2, ScaffMatch, and Boss to produce scaffolds. Score every scaffold output against the reference genome, computing F-score and error counts, and compile these into per-genome F-score/error tables. Additionally compute average contig read-coverage for each aligner-built graph and derive pairwise Pearson correlations against Fast-SG's values as a coverage-consistency check. Assemble all tables into a single benchmark results file with one row per genome-scaffolder combination; Fast-SG consistently reached better scaffolding results than the aligner-built graphs across the four real cases.

## Related tools

- **Fast-SG** (Alignment-free construction of the scaffolding graph (k-mer sizes 12-256 via FAST-SG.pl) whose scaffold outputs set the benchmark baseline against aligner-built graphs) — https://github.com/adigenova/fast-sg
- **PERL** (Wrapping the Fast-SG pipeline (FAST-SG.pl) used to generate the graphs being benchmarked) — https://github.com/adigenova/fast-sg
- **C++** (Implementation language of Fast-SG whose graph-construction performance is scored in the F-score/error benchmark) — https://github.com/adigenova/fast-sg

## Evaluation signals

- The F-score/error table contains rows for all four genomes x five graph-construction methods (Fast-SG plus Bowtie, Bowtie2, BWA, BWA-MEM)
- The Pearson correlation column contains a numeric coefficient for every aligner comparison against Fast-SG's contig read-coverage values
- The final benchmark results file has exactly one row per genome-scaffolder combination (Opera-LG, Besst2, ScaffMatch, Boss)
- Fast-SG-built graphs yield consistently equal or higher F-scores (and equal or lower error counts) than the four aligner-built graphs across the four real test cases
- Scaffolds produced by short-read scaffolders from Fast-SG synthetic libraries match or exceed those obtained by Links, a dedicated hybrid long-read scaffolder

## Limitations

- Evaluation is reference-based: without a high-quality reference genome, F-score and error counts cannot be computed
- Coverage correlations compare contig read-coverage of aligner-built graphs against Fast-SG, so graph-coverage differences may reflect aligner mapping biases rather than scaffolding quality alone
- Results were established on four real test cases and four scaffolders; extrapolation to other genomes, library types, or scaffolders is untested in this benchmark

## Evidence

- [abstract] Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [discussion] Scaffolders fed with Fast-SG synthetic libraries match Links: "short-read scaffolders are able to produce accurate scaffolds when they are fed with the synthetic libraries extracted by Fast-SG"
- [results] Consistent improvement over aligner-built graphs: "Fast-SG consistently reached better scaffolding results than the short-read aligners evaluated"
- [readme] Fast-SG is an ultrafast alignment-free scaffolding graph builder: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [methods] k-mer size parameter range for graph construction: [evidence span withheld — anchor status: not_found]
