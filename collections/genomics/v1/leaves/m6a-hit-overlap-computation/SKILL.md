---
name: m6a-hit-overlap-computation
description: Use when dRNA-seq replicates were analyzed both as separate samples and
  merged, and you need to quantify per-tool overlap of called m6A bins between the
  two configurations.
when_to_use_negative:
- Tools that cannot process multiple samples (single-sample tools), since merging
  vs separating replicates is not meaningful for them
- Datasets without biological replicates, where a separate-vs-merged comparison cannot
  be constructed
- Comparing hits across different genomes/annotations or threshold sets, since overlap
  is only interpretable when pipeline.conf settings are identical between runs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that runs the m6A detection tools under both replicate configurations
    and provides the R post-processing and statistical analysis scripts used to extract
    and overlap m6A hit bins
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine used to execute NanOlympicsMod via 'nextflow -c pipeline.conf
    run pipeline.nf' for both configurations
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Yanocomp
  role: Multi-sample, replicate-capable m6A detection tool run under both configurations;
    hits filtered at its FDR threshold
  repo: https://github.com/mfurla/NanOlympicsMod
- name: ELIGOS
  role: Multi-sample, replicate-capable m6A detection tool run under both configurations
    whose hit-bin overlap is computed
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nanocompore
  role: Multi-sample, replicate-capable m6A detection tool run under both configurations
    whose hit-bin overlap is computed
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:18d51900cdf692359f15d41637e4700ed36fbf6cd952536d2a82355377fa87cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - ELIGOS
  - NanOlympicsMod
  - Nanocompore
  - Nextflow
  - Yanocomp
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# m6a-hit-overlap-computation

## Summary

Computes per-tool pairwise overlap of m6A hit bins between two configurations of replicate-capable dRNA-seq m6A detection tools (separate replicates vs merged replicates), using NanOlympicsMod's R post-processing scripts and Jaccard-style bin overlap to quantify how much the two configurations agree.

## When to use

Use when dRNA-seq replicates were analyzed both as separate samples and merged, and you need to quantify per-tool overlap of called m6A bins between the two configurations.

## When NOT to use

- Tools that cannot process multiple samples (single-sample tools), since merging vs separating replicates is not meaningful for them
- Datasets without biological replicates, where a separate-vs-merged comparison cannot be constructed
- Comparing hits across different genomes/annotations or threshold sets, since overlap is only interpretable when pipeline.conf settings are identical between runs

## Inputs

- Yeast SK1 Nanopore dRNA-seq fast5 base-called data (three replicates)
- Tab-separated samples file (sample name, condition, path to base-called fast5 folder)
- pipeline.conf with --samples, --test_condition, SK1 genome/transcriptome fasta and index files, genes-to-transcripts and bed annotation files, per-tool thresholds (yanocomp FDR, drummer p-value, differr FDR), 'default' m6A-site filtering threshold set
- NanOlympicsMod pipeline outputs for both configurations (separate replicates and merged replicates)

## Outputs

- Per-tool number of m6A hit bins for each configuration (separate vs merged)
- Per-tool pairwise overlap percentage of m6A hits between the two configurations (Jaccard-style overlap of called bins)
- Summary table with per-tool hit counts and overlap percentages

## How to apply

Run the replicate-capable multi-sample tools (Yanocomp, ELIGOS, DiffErr, Nanocompore) through the NanOlympicsMod Nextflow pipeline twice: once with the three yeast SK1 replicates listed as separate entries in the --samples tab-separated file, and once with replicates merged into a single sample entry, keeping pipeline.conf settings (genome fasta, genes-to-transcripts and bed annotations, per-tool thresholds such as yanocomp FDR, drummer p-value, differr FDR, and the 'default' m6A-site filtering threshold set) identical between runs. Post-process both runs with the repository's R post-processing and statistical analysis scripts to extract, per tool, the number of m6A hit bins in each configuration. Then compute, per tool, the pairwise Jaccard-style overlap of called bins between the separate-replicates and merged configurations and assemble a summary table with per-tool hit counts and overlap percentages. Rationale: merging changes effective coverage, so comparing hit sets tests whether replicate handling materially changes m6A calls.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that runs the m6A detection tools under both replicate configurations and provides the R post-processing and statistical analysis scripts used to extract and overlap m6A hit bins) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine used to execute NanOlympicsMod via 'nextflow -c pipeline.conf run pipeline.nf' for both configurations) — https://github.com/mfurla/NanOlympicsMod
- **Yanocomp** (Multi-sample, replicate-capable m6A detection tool run under both configurations; hits filtered at its FDR threshold) — https://github.com/mfurla/NanOlympicsMod
- **ELIGOS** (Multi-sample, replicate-capable m6A detection tool run under both configurations whose hit-bin overlap is computed) — https://github.com/mfurla/NanOlympicsMod
- **Nanocompore** (Multi-sample, replicate-capable m6A detection tool run under both configurations whose hit-bin overlap is computed) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- For every replicate-capable tool, the overlap between the separate-replicates and merged configurations exceeds 59%
- Four of the six replicate-capable tools show overlaps greater than 99%, reproducing the concordance pattern reported in Supplementary Figure S5
- Per-tool hit counts are successfully extracted for both configurations by the repository's R post-processing scripts
- The hits of the configuration with fewer methylated sites are typically a subset of the other configuration's hits
- Both runs used identical pipeline.conf settings (same fasta/index, annotations, per-tool thresholds, 'default' threshold set), differing only in the samples file

## Limitations

- Merging replicates affected most replicate-capable tools, so hit counts differ between configurations even when overlap is high
- Several tools could not complete the benchmark runs; for a few, runs were abandoned due to lack of progress or unexpected empty output files
- The yeast dataset lacks a ground truth for m6A positions; overlap between configurations does not validate biological correctness, only concordance
- High computational burden: the full benchmark data comprise >22 M files totalling 4 TB of raw data

## Evidence

- [results] Merging replicates affected most replicate-capable tools, but the hits of the configuration with fewer methylated sites were typically a subset of the other: [evidence span withheld — anchor status: not_found]
- [readme] Samples file and pipeline configuration: "--samples                                                Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder"
- [readme] Pipeline execution command: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
- [discussion] Multi-sample tools performed better on yeast: "The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset, probably benefiting from the higher coverage."
- [methods] Threshold filtering options: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
