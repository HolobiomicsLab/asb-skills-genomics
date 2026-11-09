---
name: rank-stabilization-median-permutation
description: Use when reproducing the VIBE benchmark (305 patient cases / 308 causal
  genes) or any gene-prioritization benchmark where the causal gene may be missing
  from tool output and its rank must be randomly assigned, requiring outlier-robust
  per-case rank aggregation.
when_to_use_negative:
- When the causal gene is always present in every tool's returned list, so no random-rank
  assignment (and hence no permutation stabilization) is needed
- When analyzing the VIBE tool's actual runtime behavior rather than reproducing the
  benchmark figures — use the standard VIBE workflow with the HDT database and HPO.owl
  instead
tools:
- name: Python
  role: Used with pandas to load per-case ranked gene lists, apply the random-rank
    rule for absent genes, compute the median of 25 permutations, and write per-rank
    counts and unique-hit counts to CSV
  repo: https://github.com/molgenis/vibe-suppl
- name: R
  role: Used for benchmarking results processing and plotting (PaperPlots.R generates
    the paper figures from the stabilized rank data)
  repo: https://github.com/molgenis/vibe-suppl
- name: VIBE
  role: Source of the per-case prioritized gene lists whose causal-gene ranks are
    stabilized and compared against seven other phenotype-prioritization tools
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d2b5bdc668b08aef9b57c3085429098477deba51da3ea03e6a54e8e6007cd23f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe-suppl
  tools:
  - Python
  - R
  - VIBE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rank-stabilization-median-permutation

## Summary

A benchmark post-processing technique from the VIBE paper that stabilizes a tool's causal-gene rank by taking the median rank over 25 permutations, compensating for outliers introduced when genes absent from a tool's output must be assigned a random rank positioned after the returned genes.

## When to use

Use when reproducing the VIBE benchmark (305 patient cases / 308 causal genes) or any gene-prioritization benchmark where the causal gene may be missing from tool output and its rank must be randomly assigned, requiring outlier-robust per-case rank aggregation.

## When NOT to use

- When the causal gene is always present in every tool's returned list, so no random-rank assignment (and hence no permutation stabilization) is needed
- When analyzing the VIBE tool's actual runtime behavior rather than reproducing the benchmark figures — use the standard VIBE workflow with the HDT database and HPO.owl instead

## Inputs

- Per-case ranked gene lists for the 308 simulated clinical-exome panels (e.g., vibe_results.tsv with HGNC symbols converted to NCBI gene IDs)
- List of known causal gene identifiers (NCBI gene IDs) per patient case
- Conversion file mapping HGNC gene symbols to NCBI gene IDs (benchmark_file_conversion_data.tsv from genenames.org)

## Outputs

- Per-case stabilized causal-gene ranks (median of 25 permutations)
- Per-rank-position counts of causal genes ranked 1 through 20 and cumulative top-20 recall curve
- Unique top-20 hit counts per tool (e.g., vibe_top20_recall_unique_hits.csv)

## How to apply

For each of the 308 causal genes simulated into 20-gene clinical-exome panels (causal gene spiked into 19 pseudorandom CGD genes drawn from a pool of 3986 CGD NCBI identifiers), collect the tool's ranked gene list. When the causal gene is absent from the tool output, assign it a random rank positioned after the returned genes, since in practice the causal gene is found between rank 16 and 20 in that scenario. Because this random-rank assignment is stochastic, run 25 permutations and set each case's final rank to the median of the 25 values, which compensates for outliers. Then count per rank position how many causal genes are ranked first, second, third, up to rank 20, derive cumulative top-20 recall, and compute unique top-20 hits (e.g., VIBE uniquely prioritized 6 causal genes within the top 20, exceeded only by Phenomizer at 15 and AMELIE at 17).

## Related tools

- **Python** (Used with pandas to load per-case ranked gene lists, apply the random-rank rule for absent genes, compute the median of 25 permutations, and write per-rank counts and unique-hit counts to CSV) — https://github.com/molgenis/vibe-suppl
- **R** (Used for benchmarking results processing and plotting (PaperPlots.R generates the paper figures from the stabilized rank data)) — https://github.com/molgenis/vibe-suppl
- **VIBE** (Source of the per-case prioritized gene lists whose causal-gene ranks are stabilized and compared against seven other phenotype-prioritization tools) — https://github.com/molgenis/vibe

## Examples

```
stabilized = df.groupby('case')['rank'].median()  # median of 25 permutations after random-rank assignment for absent causal genes
```

## Evaluation signals

- Cumulative top-20 recall reaches the full total recall of 308 causal genes at rank 20 (as reported: 'At rank 20, all tools arrive at their total recall of 308 causal genes')
- VIBE's unique top-20 hit count equals the reported value of 6, with only Phenomizer (15) and AMELIE (17) exceeding it
- Number of missed genes (causal gene absent from tool output) matches Figure 1's labels for each tool
- Per-case ranks are stable: the median of 25 permutations no longer changes across additional permutation seeds
- Recomputed per-rank counts reproduce the paper's top-20 recall curve when run against the Zenodo benchmark files (record 3662470, PaperPlots.R)

## Limitations

- The random rank assigned to absent causal genes is inherently stochastic; without median-of-25 stabilization, individual cases can be extreme outliers
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- PhenoTips benchmarking is obsolete (stand-alone download no longer available; paid cloud subscription required), so the full seven-tool comparison cannot be rerun as published

## Evidence

- [methods] To compensate for outliers, we stabilized the rank as the median of 25 permutations: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [methods] For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene: "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [results] At rank 20, all tools arrive at their total recall of 308 causal genes.: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [results] Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list.: "Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list."
- [readme] Please refer to the `README.md` at https://zenodo.org/record/3662470 for the exact commits used for the benchmarking.: [evidence span withheld — anchor status: not_found]
