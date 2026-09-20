---
name: replicate-merge-configuration-comparison
description: Use when replicate-capable m6A tools (ELIGOS, Yanocomp, DiffErr, Nanocompore,
  drummer, differr) are run on multi-replicate dRNA-seq data and you need to know
  whether merging replicates alters the detected m6A site set, or how the two configurations
  overlap.
when_to_use_negative:
- The dataset has only a single replicate per condition, so there is no separate-versus-merged
  choice to compare.
- The tools of interest are not replicate-capable multi-sample tools (e.g. single-sample
  tools cannot exploit the replicate configuration difference).
- You lack the base-called fast5 folder structure and SK1 genome/transcriptome annotation
  required by the NanOlympicsMod pipeline.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0085
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow pipeline used to run the m6A detection tools on both
    replicate configurations and produce per-tool hit outputs
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing pipeline.nf with the pipeline.conf configuration
    for each replicate configuration
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Yanocomp
  role: Replicate-capable multi-sample m6A detection tool run on both configurations
    with its FDR threshold set in pipeline.conf
- name: ELIGOS
  role: Replicate-capable multi-sample m6A detection tool compared across separate
    and merged replicate configurations
- name: Nanocompore
  role: Replicate-capable multi-sample m6A detection tool included in the separate-versus-merged
    comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:c5a662f37e1b090c5708c56766cd423e7268b349f4c06b4a08589315649af7ce
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# replicate-merge-configuration-comparison

## Summary

A method for testing how presenting replicate Nanopore dRNA-seq data as separate samples versus a single merged sample changes the m6A hits reported by replicate-capable multi-sample tools, and quantifying the per-tool overlap between the two configurations using the NanOlympicsMod Nextflow pipeline.

## When to use

Use when replicate-capable m6A tools (ELIGOS, Yanocomp, DiffErr, Nanocompore, drummer, differr) are run on multi-replicate dRNA-seq data and you need to know whether merging replicates alters the detected m6A site set, or how the two configurations overlap.

## When NOT to use

- The dataset has only a single replicate per condition, so there is no separate-versus-merged choice to compare.
- The tools of interest are not replicate-capable multi-sample tools (e.g. single-sample tools cannot exploit the replicate configuration difference).
- You lack the base-called fast5 folder structure and SK1 genome/transcriptome annotation required by the NanOlympicsMod pipeline.

## Inputs

- Yeast SK1 Nanopore dRNA-seq fast5 data (three replicates)
- Tab-separated samples file (sample name, condition, path to base-called fast5 folder)
- SK1 reference genome and transcriptome fasta files plus index files
- Genes-to-transcripts and bed annotation files
- pipeline.conf with per-tool significance thresholds and 'default' m6A-site filtering threshold set

## Outputs

- Per-tool m6A hit bin lists for the separate-replicates configuration
- Per-tool m6A hit bin lists for the merged-replicate configuration
- Summary table with per-tool hit counts per configuration and pairwise overlap percentages (Jaccard-style overlap of called bins)

## How to apply

Retrieve the yeast SK1 Nanopore dRNA-seq fast5 data and build a tab-separated samples file listing sample name, condition, and path to the base-called fast5 folder, as required by the NanOlympicsMod --samples argument. Configure pipeline.conf in the cloned NanOlympicsMod repository with --samples, --test_condition, the SK1 genome/transcriptome fasta and index files, genes-to-transcripts and bed annotation files, per-tool significance thresholds (yanocomp FDR threshold, drummer p-value threshold, differr FDR threshold), and the 'default' m6A-site filtering threshold set. Run configuration A with 'nextflow -c pipeline.conf run pipeline.nf' providing the three replicates as separate sample entries, then run configuration B with the replicates merged into one sample entry, keeping all other settings identical. Post-process both runs with the repository's R post-processing and statistical analysis scripts to extract per-tool m6A hit bin counts, then compute pairwise overlap of called bins between configurations and assemble a summary table with per-tool hit counts and overlap percentages.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow pipeline used to run the m6A detection tools on both replicate configurations and produce per-tool hit outputs) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing pipeline.nf with the pipeline.conf configuration for each replicate configuration) — https://github.com/mfurla/NanOlympicsMod
- **Yanocomp** (Replicate-capable multi-sample m6A detection tool run on both configurations with its FDR threshold set in pipeline.conf)
- **ELIGOS** (Replicate-capable multi-sample m6A detection tool compared across separate and merged replicate configurations)
- **Nanocompore** (Replicate-capable multi-sample m6A detection tool included in the separate-versus-merged comparison)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- For every replicate-capable tool, the overlap between separate-replicates and merged configurations exceeds 59%.
- Four out of six replicate-capable tools show overlaps greater than 99%, reproducing the Supplementary Figure S5 concordance pattern.
- The configuration with fewer methylated sites typically yields hits that are a subset of the other configuration's hits.
- Per-tool hit bin counts are successfully extracted for both configurations via the repository's R post-processing scripts.
- The summary table is complete with one hit-count pair and one overlap percentage per tool.

## Limitations

- Yeast, mouse and human datasets lack a ground truth for m6A positions; reference sets are derived from orthogonal short-read methods, so overlap concordance does not equal correctness.
- Several tools could not complete the benchmark runs (lack of progress or empty output files), so per-tool results may be incomplete.
- Merging replicates reduces the number of sample-level comparisons available to multi-sample tools, which may interact with per-tool significance thresholds.

## Evidence

- [readme] Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder: "Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder"
- [discussion] Multi-sample tools performed better in yeast dataset: "The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset"
- [abstract] NanOlympicsMod pipeline description: "a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data"
- [methods] NanOlympicsMod repository installation: "git clone https://github.com/mfurla/NanOlympicsMod.git"
- [readme] Post-processing scripts in repository: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
