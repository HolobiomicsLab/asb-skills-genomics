---
name: single-copy-marker-gene-bin-scoring
description: Use when candidate bins (e.g. from per-epsilon DBSCAN runs on long-read
  contig embeddings) must be ranked by quality to drive greedy ensemble bin selection
  in SemiBin2.
when_to_use_negative:
- Binning short-read data, where SemiBin2 uses its standard single-clustering path
  rather than the ensemble DBSCAN with marker-gene-based greedy selection
- When reference-based quality assessment (e.g. against ground-truth genomes in a
  benchmark) is available and marker-gene-based estimates would be redundant
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: SemiBin2
  role: Implements the DBSCAN ensemble and greedy Algorithm 1 that consumes the marker-gene-derived
    completeness/contamination/F1 scores
  repo: https://github.com/BigDataBiology/SemiBin/
- name: hmmer
  role: External tool used to detect single-copy marker gene occurrences for estimating
    bin completeness and contamination
- name: DBSCAN
  role: Clustering (via scikit-learn) that generates the candidate bins being scored
    at 12 epsilon values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:336827a2113d8b00ed3ab20b7b3851e6c6d540825dde3bdd2720d9ca6cdbc0cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - DBSCAN
  - SemiBin2
  - hmmer
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Single-copy marker gene bin scoring

## Summary

Scores candidate metagenomic bins by estimating completeness and contamination from 107 prokaryotic single-copy marker genes (e.g. with hmmer) and computing an F1 score, enabling SemiBin2's greedy ensemble selection of DBSCAN-generated bins on long-read data.

## When to use

Use when candidate bins (e.g. from per-epsilon DBSCAN runs on long-read contig embeddings) must be ranked by quality to drive greedy ensemble bin selection in SemiBin2.

## When NOT to use

- Binning short-read data, where SemiBin2 uses its standard single-clustering path rather than the ensemble DBSCAN with marker-gene-based greedy selection
- When reference-based quality assessment (e.g. against ground-truth genomes in a benchmark) is available and marker-gene-based estimates would be redundant

## Inputs

- Candidate bin sets from DBSCAN runs at 12 epsilon values (0.01–0.55) on SemiBin2 learned contig embeddings
- Long-read contigs used for binning (--sequencing-type=long_read)
- Deep learning feature files (data.csv / data_split.csv)
- Reference set of 107 prokaryotic single-copy marker genes

## Outputs

- Per-bin completeness and contamination estimates
- Per-bin F1 scores used for greedy ensemble selection
- Final ensemble bins as gzipped FASTA files (one per bin) in output_bins
- bins_info.tsv (bin quality summary) and contig_bins.tsv (contig-to-bin assignment)

## How to apply

For each candidate bin produced by DBSCAN on the learned contig embeddings (data.csv/data_split.csv features from --sequencing-type=long_read), identify occurrences of the 107 prokaryotic single-copy marker genes, e.g. by gene prediction and searching with hmmer. From marker gene copy counts estimate completeness (fraction of markers present) and contamination (excess copies), then compute the F1 score per Eq. 4 of the SemiBin2 paper. Feed these scores into the greedy Algorithm 1: iteratively accept the highest-F1 bin whose contamination does not exceed the threshold, sweeping thresholds from 0.1 to 1.0 and removing contigs already assigned to an accepted bin, until no qualifying bins remain. The rationale is that single-copy genes provide an assembly-free, reference-free proxy for bin completeness/contamination that can rank bins from different clustering runs on a common scale.

## Related tools

- **SemiBin2** (Implements the DBSCAN ensemble and greedy Algorithm 1 that consumes the marker-gene-derived completeness/contamination/F1 scores) — https://github.com/BigDataBiology/SemiBin/
- **hmmer** (External tool used to detect single-copy marker gene occurrences for estimating bin completeness and contamination)
- **DBSCAN** (Clustering (via scikit-learn) that generates the candidate bins being scored at 12 epsilon values)

## Examples

```
SemiBin single_easy_bin -i contigs.fasta -b sample.bam --sequencing-type=long_read -o output_bins
```

## Evaluation signals

- Every candidate bin has non-empty completeness and contamination values derived from the 107 single-copy marker genes and a computed F1 score
- Accepted bins satisfy the contamination threshold being swept (0.1–1.0), and no accepted bin shares contigs with a previously accepted bin
- The final ensemble bin set yields more high-quality bins than any single-epsilon DBSCAN run
- Output directory contains gzipped FASTA per bin plus bins_info.tsv and contig_bins.tsv with consistent contig identifiers

## Limitations

- Marker-gene-based completeness/contamination are estimates; bins lacking marker genes (e.g. small or divergent genomes) may be scored unreliably and filtered out by the greedy algorithm
- The ensemble DBSCAN plus scoring path is specific to long-read binning in SemiBin2; results depend on the epsilon range (0.01–0.55) and contamination thresholds swept
- SemiBin operations are probabilistic; running twice may return different candidate bins and hence different scores

## Evidence

- [methods] scores every bin with 107 single-copy marker genes (completeness, contamination, F1-score): [evidence span withheld — anchor status: not_found]
- [abstract] ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [methods] external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan): "the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan)"
- [readme] Many of the operations of SemiBin are probabilistic and running it twice will return different results: [evidence span withheld — anchor status: external_doc]
