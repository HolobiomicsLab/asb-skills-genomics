---
name: overlap-graph-quality-assessment
description: Use when a candidate overlap graph from an overlap detection tool must
  be judged against truth/reference overlaps on long-read datasets, e.g. comparing
  Fedrann weighting x metric configurations or tools like minimap2 and MECAT2.
when_to_use_negative:
- Input is a feature matrix or dimensionality-reduced embedding rather than a set
  of candidate overlaps — evaluation requires a concrete overlap graph
- No reference truth overlaps are available (e.g. unassembled novel sample with no
  ground-truth mapping), so error rates cannot be computed
- The goal is end-to-end assembly quality (contiguity, BUSCO) rather than overlap
  graph accuracy — use assembler-level metrics instead
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Produces the candidate overlap graphs (overlaps.tsv) that are scored against
    truth overlaps
  repo: https://github.com/jzhang-dev/fedrann
- name: minimap2
  role: Generates reference read alignments (PAF) from which truth overlaps are derived
    after filtering
- name: kNN-overlap-finder
  role: Contains the reproducible benchmarking and evaluation workflows used to score
    overlap graphs
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:cd8e2a4c5a3d3b7812ec446294a83ae33d4a6fd29252d89fe8e61534baa4591d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
  tools:
  - Fedrann
  - kNN-overlap-finder
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# overlap-graph-quality-assessment

## Summary

Scores overlap graphs (e.g. Fedrann's overlaps.tsv) against reference truth overlaps by applying alignment filters (>= 100 matched bases, >= 10% identity) and computing overlap error rates, enabling benchmark comparison across tool or parameter configurations.

## When to use

Use when a candidate overlap graph from an overlap detection tool must be judged against truth/reference overlaps on long-read datasets, e.g. comparing Fedrann weighting x metric configurations or tools like minimap2 and MECAT2.

## When NOT to use

- Input is a feature matrix or dimensionality-reduced embedding rather than a set of candidate overlaps — evaluation requires a concrete overlap graph
- No reference truth overlaps are available (e.g. unassembled novel sample with no ground-truth mapping), so error rates cannot be computed
- The goal is end-to-end assembly quality (contiguity, BUSCO) rather than overlap graph accuracy — use assembler-level metrics instead

## Inputs

- overlaps.tsv candidate overlap files (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- Reference truth overlap sets derived from read alignments (e.g. minimap2 PAF files)
- Long-read sequencing input in FASTA/FASTQ form (e.g. test/data/reads.fasta.gz)
- Reference genome assemblies such as T2T-CHM13 v2.0, WBcel235 (GCA_000002985.3), or Drosophila melanogaster Release 6 (GCF_000001215.4)

## Outputs

- Per-configuration overlap error rates
- A comparison table of overlap error rates across tool/parameter combinations (e.g. 8 weighting x distance configurations)
- Filtered truth overlap sets meeting >= 100 matched bases and >= 10% identity

## How to apply

Collect candidate overlaps from each tool run (e.g. Fedrann's overlaps.tsv, which lists query/target names, orientations, neighbor_rank, and distance). Score every candidate overlap graph against reference truth overlaps derived from read alignments; when computing truth, first filter reference reads (length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30) and then filter alignments by two criteria: >= 100 matched bases and >= 10% alignment identity (raise the identity threshold to 30% for minimap2 PAF files on large human alignments). Classify each predicted overlap as true or false under these thresholds and compute overlap error rates per configuration. Assemble error rates across all configurations (e.g. 8 weighting x distance-metric combinations such as TF/IDF/TF-IDF/raw x cosine/Euclidean) into a single comparison table. Validate that the expected best configuration (IDF weighting with cosine distance, consistent with the IDF + SRP + NNDescent pipeline) exhibits the lowest error rate.

## Related tools

- **Fedrann** (Produces the candidate overlap graphs (overlaps.tsv) that are scored against truth overlaps) — https://github.com/jzhang-dev/fedrann
- **minimap2** (Generates reference read alignments (PAF) from which truth overlaps are derived after filtering)
- **kNN-overlap-finder** (Contains the reproducible benchmarking and evaluation workflows used to score overlap graphs) — https://github.com/jzhang-dev/kNN-overlap-finder

## Evaluation signals

- Every predicted overlap in overlaps.tsv is classified against a filtered truth set (>= 100 matched bases, >= 10% identity), and no predictions remain unclassified
- The reference truth set only contains reads passing length >= 5 kb, aligned fraction >= 50%, and mapping quality >= 30
- Error rates are computed for every configuration in the grid (e.g. all 8 weighting x distance combinations) and assembled into one comparison table
- The IDF + cosine configuration shows the lowest error rate, consistent with the reported IDF + SRP + NNDescent pipeline accuracy
- Fedrann's overlap graph error rates are comparable to or lower than MECAT2, minimap2, and wtdbg2 baselines

## Limitations

- For minimap2 PAF files producing exceptionally large outputs (exceeding 4TB for human whole-genome alignments), the identity threshold must be raised to 30%, which changes truth-set strictness between tools
- Large-scale truth computation is memory- and I/O-intensive; minimap2 human PAF outputs can exceed 4TB
- Reproducibility depends on the GitHub repositories (kNN-overlap-finder); no DOI or Zenodo archive is provided

## Evidence

- [discussion] All alignments were filtered using two criteria: (1) $≥ 100 matched bases and (2) $≥ 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [discussion] read length $≥ 5 kb; (2) aligned fraction $≥ 50%; (3) mapping quality $≥ 30: [evidence span withheld — anchor status: not_found]
- [readme] This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics.: "This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics."
- [abstract] Fedrann produces overlap graphs comparable to or better than those generated by existing state-of-the-art tools, including MECAT2, minimap2, and wtdbg2: "Fedrann produces overlap graphs comparable to or better than those generated by existing state-of-the-art tools, including MECAT2, minimap2, and wtdbg2, while maintaining competitive runtime"
- [discussion] Reproducible workflows for benchmarking and evaluation: https://github.com/jzhang-dev/kNN-overlap-finder: "Reproducible workflows for benchmarking and evaluation: https://github.com/jzhang-dev/kNN-overlap-finder"
