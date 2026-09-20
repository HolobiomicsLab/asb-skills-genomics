---
name: runtime-memory-profiling
description: Use when benchmarking overlap-detection or dimensionality-reduction methods
  and you need to record wall-clock runtime and peak memory per method to judge which
  scale within constraints such as a 6-hour / 1-TB budget.
when_to_use_negative:
- When the goal is overlap-graph accuracy only and no resource budget must be met
  — accuracy metrics alone suffice.
- When inputs are already aggregated benchmark tables without per-run logs, since
  runtime and peak memory cannot be re-derived.
- When evaluating a single configuration with no comparative baseline to profile against.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Fedrann
  role: Produces the fedrann.log from which wall-clock runtime and peak memory per
    run are recorded
  repo: https://github.com/jzhang-dev/fedrann
- name: kNN-overlap-finder
  role: Reproducible benchmarking workflows that record runtime and peak memory for
    each baseline tool run
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: minimap2
  role: Baseline overlap detection tool whose runtime and peak memory are profiled
    under the same conditions
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:d21ff36caf3d2ddc356f7c7c264e2ec034a757cfba994e5fbfef334e3ff85ebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# runtime-memory-profiling

## Summary

Profiles wall-clock runtime and peak memory consumption per tool or pipeline stage from log output (e.g. fedrann.log) to compare computational efficiency of methods like SRP, GRP, PCA, and UMAP under fixed resource budgets.

## When to use

Use when benchmarking overlap-detection or dimensionality-reduction methods and you need to record wall-clock runtime and peak memory per method to judge which scale within constraints such as a 6-hour / 1-TB budget.

## When NOT to use

- When the goal is overlap-graph accuracy only and no resource budget must be met — accuracy metrics alone suffice.
- When inputs are already aggregated benchmark tables without per-run logs, since runtime and peak memory cannot be re-derived.
- When evaluating a single configuration with no comparative baseline to profile against.

## Inputs

- fedrann.log output from each Fedrann run (wall-clock runtime and peak memory)
- Long-read sequencing datasets (e.g. ONT SRR10028111, SRR13070625, PacBio HiFi HG002, H3C/H3O/H3P, H4C/H4O/H4P)
- Runtime and peak-memory measurements from baseline tools (minimap2, MECAT2, MHAP, wtdbg2, xRead, BLEND)

## Outputs

- Per-method wall-clock runtime and peak-memory table (one row per tool-dataset combination)
- Combined CSV with columns for error rate, mean overlap size, #COC, #CC, runtime, and peak memory
- Scalability verdict (which methods complete within predefined resource constraints)

## How to apply

Run each method (e.g. the DR methods PCA, UMAP, Spectral Embedding, scBiMapping, GRP, SimHash, and SRP fed IDF-weighted k-mer features, or the overlap tools MECAT2, minimap2, wtdbg2, MHAP, BLEND, xRead, and Fedrann) on the same read sets with a fixed random seed and multi-threaded execution for reproducibility. For Fedrann runs, record wall-clock runtime and peak memory consumption per method from the fedrann.log output; for baselines, use the reproducible benchmarking workflows in github.com/jzhang-dev/kNN-overlap-finder. Aggregate the resource metrics alongside accuracy metrics (error rate, #COC, #CC) into a comparison table with one row per tool-dataset combination. Use the profiles to decide scalability: in this work only SRP, GRP, and SimHash completed on the largest H3-class datasets (H3C, H3O, H3P), with SRP fastest and lowest-memory. Validate a comparison only if every method row contains both accuracy and resource measurements and the resource budget check (e.g. SRP runtime under 6 hours, peak memory under 1 TB on the largest dataset) passes.

## Related tools

- **Fedrann** (Produces the fedrann.log from which wall-clock runtime and peak memory per run are recorded) — https://github.com/jzhang-dev/fedrann
- **kNN-overlap-finder** (Reproducible benchmarking workflows that record runtime and peak memory for each baseline tool run) — https://github.com/jzhang-dev/kNN-overlap-finder
- **minimap2** (Baseline overlap detection tool whose runtime and peak memory are profiled under the same conditions)

## Examples

```
fedrann -i reads.fasta.gz -o out/ -k 30 --kmer-sample-fraction 0.05 --kmer-min-multiplicity 2 --seed 602 --threads 32; grep -E 'runtime|memory' out/fedrann.log
```

## Evaluation signals

- Every method row in the comparison table contains both accuracy and resource measurements.
- SRP's runtime is under 6 hours with peak memory under 1 TB on the largest H3-class dataset.
- The aggregated CSV has one row per tool-dataset combination (7 tools x 5 datasets) with all required metric fields populated.
- Fixed random seed and multi-threaded execution are used so resource measurements are reproducible.
- Metrics are sourced from fedrann.log for Fedrann runs rather than estimated.

## Limitations

- Memory consumption of Fedrann is higher than most existing overlap detection tools, limiting comparability on memory-constrained systems.
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive of the benchmarking setup is provided.
- Runtime/memory comparisons are valid only when seeds, thread counts, and hardware are held fixed across methods.

## Evidence

- [methods] Record wall-clock runtime and peak memory consumption per method from the fedrann.log output, using a fixed random seed and multi-threaded execution for reproducibility.: [evidence span withheld — anchor status: not_found]
- [results] SRP fastest with lowest memory among scaled methods.: [evidence span withheld — anchor status: not_found]
- [readme] fedrann.log details pipeline progress.: "fedrann.log: A log file that details the pipeline's progress, from start to finish."
- [readme] Uniform benchmarking workflows record runtime and memory for each tool.: [evidence span withheld — anchor status: external_doc]
