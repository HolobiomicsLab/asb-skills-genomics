---
name: overlap-graph-connectivity-metrics-computation
description: Use when you have overlap outputs (e.g. overlaps.tsv or PAF/MHAP format)
  from one or more overlap-detection tools and need to compare overlap graph quality
  across tools or datasets using ground-truth alignments.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ/FASTA) rather than a candidate overlap set
  from a detection tool
- The goal is assembly-quality assessment (e.g. BUSCO/primates_odb10 completeness,
  contiguity of Fedrann-Shasta assemblies) rather than overlap graph quality
- The application is a downstream use of overlaps (e.g. metagenomic binning, taxonomic
  classification) which the authors note lacks empirical validation of these metrics
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0092
tools:
- name: kNN-overlap-finder
  role: Provides the reproducible benchmarking and evaluation workflows used to compute
    the overlap-graph metrics for all tool-dataset pairs
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: minimap2
  role: Produces reference-based ground-truth PAF alignments that are filtered to
    define truth overlaps; identity threshold raised to 30% for large human PAF outputs
- name: Fedrann
  role: Tool under evaluation; its overlaps.tsv output (query/target names, orientations,
    neighbor_rank, distance) is scored against ground truth
  repo: https://github.com/jzhang-dev/FEDRANN
- name: MHAP
  role: Baseline overlap detection tool whose output overlaps are benchmarked with
    the same evaluation metrics
- name: MECAT2
  role: Baseline overlap detection tool; slightly outperformed Fedrann on the H4C
    dataset in the reference benchmark
- name: wtdbg2
  role: Baseline overlap detection tool included in the uniform benchmarking workflows
- name: xRead
  role: Baseline overlap detection tool included in the uniform benchmarking workflows
- name: BLEND
  role: Baseline overlap detection tool included in the uniform benchmarking workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:46f853404b3e3b30c2237d696911e214bab0891803f4bb8dbbd2129ac42c1d21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/kNN-overlap-finder
  tools:
  - BLEND
  - Fedrann
  - MECAT2
  - MHAP
  - Python
  - kNN-overlap-finder
  - minimap2
  - wtdbg2
  - xRead
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# overlap-graph-connectivity-metrics-computation

## Summary

Evaluates candidate overlap sets from long-read overlap detection tools (Fedrann, minimap2, MECAT2, MHAP, wtdbg2, xRead, BLEND) by scoring error rate, mean overlap size, #COC, #CC, runtime, and peak memory per tool-dataset pair against reference-based ground-truth alignments.

## When to use

Use when you have overlap outputs (e.g. overlaps.tsv or PAF/MHAP format) from one or more overlap-detection tools and need to compare overlap graph quality across tools or datasets using ground-truth alignments.

## When NOT to use

- Input is raw sequencing reads (FASTQ/FASTA) rather than a candidate overlap set from a detection tool
- The goal is assembly-quality assessment (e.g. BUSCO/primates_odb10 completeness, contiguity of Fedrann-Shasta assemblies) rather than overlap graph quality
- The application is a downstream use of overlaps (e.g. metagenomic binning, taxonomic classification) which the authors note lacks empirical validation of these metrics

## Inputs

- Candidate overlap files per tool (e.g. Fedrann overlaps.tsv, minimap2 PAF, MHAP format)
- Ground-truth read-to-reference alignments (minimap2 PAF) for real WGS datasets (C. elegans ONT SRR10028111, D. melanogaster ONT SRR13070625, human H4C/H4O/H4P from HG002)
- Wall-clock runtime and peak memory logs from each benchmarked run

## Outputs

- CSV table with one row per tool-dataset combination and columns: error rate, mean overlap size, #COC, #CC, runtime, peak memory

## How to apply

First build ground truth by aligning reads to the reference assembly (e.g. WBcel235 GCA_000002985.3, Drosophila Release 6 GCF_000001215.4, or T2T-CHM13 v2.0) and filter references by read length >= 5 kb, aligned fraction >= 50%, and mapping quality >= 30. Filter the ground-truth alignments to those with >= 100 matched bases and >= 10% alignment identity; for minimap2 PAF files on human whole-genome data, raise the identity threshold to 30% to keep outputs manageable. For each tool-dataset pair, compute error rate, mean overlap size, number of connected components (#COC), and number of connected components (#CC) from the overlap graph, and record wall-clock runtime and peak memory from the benchmark run. Aggregate all metrics into one CSV table with one row per tool-dataset combination and columns for error rate, mean overlap size, #COC, #CC, runtime, and peak memory, so results can be compared directly against the published benchmark from github.com/jzhang-dev/kNN-overlap-finder.

## Related tools

- **kNN-overlap-finder** (Provides the reproducible benchmarking and evaluation workflows used to compute the overlap-graph metrics for all tool-dataset pairs) — https://github.com/jzhang-dev/kNN-overlap-finder
- **minimap2** (Produces reference-based ground-truth PAF alignments that are filtered to define truth overlaps; identity threshold raised to 30% for large human PAF outputs)
- **Fedrann** (Tool under evaluation; its overlaps.tsv output (query/target names, orientations, neighbor_rank, distance) is scored against ground truth) — https://github.com/jzhang-dev/FEDRANN
- **MHAP** (Baseline overlap detection tool whose output overlaps are benchmarked with the same evaluation metrics)
- **MECAT2** (Baseline overlap detection tool; slightly outperformed Fedrann on the H4C dataset in the reference benchmark)
- **wtdbg2** (Baseline overlap detection tool included in the uniform benchmarking workflows)
- **xRead** (Baseline overlap detection tool included in the uniform benchmarking workflows)
- **BLEND** (Baseline overlap detection tool included in the uniform benchmarking workflows)

## Evaluation signals

- The aggregated CSV exists with all required metric fields (error rate, mean overlap size, #COC, #CC, runtime, peak memory) populated for all 7 tools x 5 datasets
- Reproduced result matches the paper's finding: Fedrann most accurate on 4/5 datasets and fastest on H4C/H4O, with MECAT2 slightly better on H4C
- Ground-truth alignments satisfy the stated filters: read length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30, >= 100 matched bases, >= 10% identity (30% identity for human minimap2 PAFs)
- Runtime and peak memory values are present for every tool-dataset row, not only accuracy metrics
- One row per tool-dataset combination with no missing combinations

## Limitations

- Reference-based evaluation depends on minimap2 PAF outputs that can exceed 4 TB for human whole-genome alignments, requiring a raised 30% identity threshold that may affect ground-truth comparability
- No DOI-archived data/zenodo snapshot; reproducibility depends solely on the GitHub repositories (jzhang-dev/FEDRANN, jzhang-dev/kNN-overlap-finder)
- Metrics validated only on the paper's five WGS datasets and read types; no validation for metagenomic or other long-read applications
- Benchmarking two tools concurrently or on different hardware invalidates runtime and peak-memory comparisons

## Evidence

- [methods] Evaluate every overlap set against reference-based ground-truth alignments (filtered by read length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30; alignments requiring >= 100 matched bases and >= 10% identity) to compute error rate, mean overlap size, number of connected components (#COC), and number of connected components (#CC) per tool-dataset pair.: [evidence span withheld — anchor status: not_found]
- [methods] Aggregate all metrics into one CSV table with one row per tool-dataset combination and columns for error rate, mean overlap size, #COC, #CC, runtime, and peak memory.: [evidence span withheld — anchor status: not_found]
- [discussion] for minimap2 PAF files (which produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%: "produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%"
- [abstract] Fedrann produces overlap graphs comparable to or better than those generated by existing state-of-the-art tools, including MECAT2, minimap2, and wtdbg2, while maintaining competitive runtime: "Fedrann produces overlap graphs comparable to or better than those generated by existing state-of-the-art tools, including MECAT2, minimap2, and wtdbg2"
- [other] Run the six baselines (minimap2, MHAP, MECAT2, wtdbg2, xRead, BLEND) on the same read sets using the reproducible benchmarking workflows from github.com/jzhang-dev/kNN-overlap-finder, recording wall-clock runtime and peak memory for each run.: [evidence span withheld — anchor status: not_found]
