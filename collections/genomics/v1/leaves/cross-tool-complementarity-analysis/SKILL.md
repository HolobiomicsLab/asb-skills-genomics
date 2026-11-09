---
name: cross-tool-complementarity-analysis
description: Use when multiple gene-prioritization tools have been run on the same
  benchmark cases and you need per-tool unique top-k hit counts and evidence of complementarity
  across tools.
when_to_use_negative:
- You have only a single tool's output — unique-hit and complementarity analysis requires
  at least two tools run on identical cases
- Tool outputs are not harmonized to a shared gene identifier space (e.g., mixed HGNC
  symbols and NCBI IDs) — the benchmark comparison relies on converting everything
  to NCBI gene identifiers first
- Cases lack known causal-gene truth labels, so per-case ranks and unique-hit counts
  cannot be computed
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3577
tools:
- name: VIBE
  role: Primary prioritization tool benchmarked; its top-20 unique hits (6) are compared
    against other tools
  repo: https://github.com/molgenis/vibe
- name: Phenomizer
  role: Comparison tool with the highest unique top-20 hit count (15) among benchmarked
    tools
- name: AMELIE
  role: Comparison tool with 17 unique top-20 causal genes, the highest unique-hit
    count in the benchmark
- name: GADO
  role: Comparison tool run as stand-alone command-line version v1.0.1 with prediction
    matrix hpo_predictions_sigOnly_spiked_01_02_2018
- name: hiPHIVE / PhenIX
  role: Exomiser-rest-prioritiser-based comparison tools (custom .jar, release 12.1.0)
    benchmarked on the same cases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:0074714f84cdeebda1ec18f136e1c7ee3b9b7d2db9fdcd88a75c8fe6912c2692
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - AMELIE
  - GADO
  - Phenomizer
  - Python
  - R
  - VIBE
  - hiPHIVE / PhenIX
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-tool-complementarity-analysis

## Summary

Quantifies how many causal genes each phenotype-based prioritization tool (e.g., VIBE, Phenomizer, AMELIE, GADO, hiPHIVE, PhenIX, PhenoTips, PubCaseFinder) uniquely recovers within a fixed rank window (top 20) over a shared benchmark, exposing complementarity among tools rather than declaring a single winner.

## When to use

Use when multiple gene-prioritization tools have been run on the same benchmark cases and you need per-tool unique top-k hit counts and evidence of complementarity across tools.

## When NOT to use

- You have only a single tool's output — unique-hit and complementarity analysis requires at least two tools run on identical cases
- Tool outputs are not harmonized to a shared gene identifier space (e.g., mixed HGNC symbols and NCBI IDs) — the benchmark comparison relies on converting everything to NCBI gene identifiers first
- Cases lack known causal-gene truth labels, so per-case ranks and unique-hit counts cannot be computed

## Inputs

- Per-case ranked gene lists for each benchmarked tool (e.g., vibe_results.tsv, amelie_results.tsv, phenomizer_results.tsv, gado_results.tsv, hiphive_results.tsv, phenix_results.tsv), converted to NCBI gene IDs
- benchmark_data.tsv (case ID + HPO phenotype names/IDs compatible with hp.obo release 2018-03-08)
- Causal-gene truth labels per case (308 causal genes from the 305 patient cases)
- benchmark_file_conversion_data.tsv (HGNC symbol to NCBI gene ID mapping from genenames.org)

## Outputs

- Per-tool top-20 causal-gene recall counts and per-rank counts
- Per-tool unique top-20 hit counts (e.g., VIBE=6, Phenomizer=15, AMELIE=17)
- CSV such as vibe_top20_recall_unique_hits.csv with per-rank counts, top-20 recall, and unique-hit counts
- Heat map (Figure 2 style) of causal-gene rank per tool per case with hierarchical clustering of tool dissimilarity

## How to apply

Run every prioritization tool on the identical benchmark input — 305 patient cases converted to HPO codes (matched to HPO release v2018-03-08), each case simulated as a clinical exome interpretation with 20 candidate genes (causal gene spiked into 19 pseudorandom CGD genes drawn from a 3986-gene CGD NCBI identifier pool). Normalize all tool outputs to NCBI gene identifiers (convert HGNC symbols via benchmark_file_conversion_data.tsv) and compute the causal gene's rank per case per tool, stabilizing ranks as the median of 25 permutations when random-rank assignment is used for genes absent from a tool's output. For each tool, count causal genes ranked within the top 20 and identify 'unique' hits — causal genes found in the top 20 by that tool alone. In the VIBE benchmark this yielded VIBE 6 unique top-20 genes, exceeded only by Phenomizer (15) and AMELIE (17), while all tools reached total recall of 308 causal genes at rank 20; complementarity was further visualized as a heat map of per-case causal-gene ranks with hierarchical clustering showing dissimilarity between tools.

## Related tools

- **VIBE** (Primary prioritization tool benchmarked; its top-20 unique hits (6) are compared against other tools) — https://github.com/molgenis/vibe
- **Phenomizer** (Comparison tool with the highest unique top-20 hit count (15) among benchmarked tools)
- **AMELIE** (Comparison tool with 17 unique top-20 causal genes, the highest unique-hit count in the benchmark)
- **GADO** (Comparison tool run as stand-alone command-line version v1.0.1 with prediction matrix hpo_predictions_sigOnly_spiked_01_02_2018)
- **hiPHIVE / PhenIX** (Exomiser-rest-prioritiser-based comparison tools (custom .jar, release 12.1.0) benchmarked on the same cases)

## Examples

```
python3 BenchmarkFileGeneSymbolToIdConverter.py vibe_results.tsv benchmark_file_conversion_data.tsv 1> vibe.log 2> vibe.err
```

## Evaluation signals

- At rank 20, each tool's cumulative recall reaches the full 308 causal genes; verify this invariant per tool
- VIBE's unique top-20 hit count must equal the reported value of 6, with Phenomizer (15) and AMELIE (17) exceeding it
- All tool result files must be converted to NCBI gene IDs without introducing duplicates or unmatched symbols (check conversion logs such as amelie.log/amelie.err)
- Per-case causal-gene ranks for tools with random-rank fallback should be stable across the 25-permutation median (outliers compensated)
- Hierarchical clustering of the per-case rank heat map should show dissimilarity between all tested tools, consistent with high complementarity

## Limitations

- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- High complementarity among all tested tools means no single tool dominates; unique-hit counts are sensitive to the top-20 cutoff and the 20-gene panel simulation
- Genes absent from a tool's output require random-rank assignment after returned genes, introducing variance that must be mediated by median-of-25-permutation stabilization
- External services such as AMELIE may offer valuable information but cannot always be used since privacy and availability are not guaranteed

## Evidence

- [results] VIBE uniquely prioritized 6 causal genes within the top 20 results, a count of uniquely detected causal genes exceeded only by Phenomizer (15) and AMELIE (17).: [evidence span withheld — anchor status: not_found]
- [results] At rank 20, all tools arrive at their total recall of 308 causal genes.: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [methods] For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene: "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [methods] To compensate for outliers, we stabilized the rank as the median of 25 permutations: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] Hierarchical clustering shows a degree of dissimilarity between all of the tested tools.: "Hierarchical clustering shows a degree of dissimilarity between all of the tested tools."
- [abstract] Our results demonstrate that VIBE offers consistent performance with few cases missed, but we also find high complementarity among all tested tools.: "Our results demonstrate that VIBE offers consistent performance with few cases missed, but we also find high complementarity among all tested tools."
- [readme] python3 BenchmarkFileGeneSymbolToIdConverter.py amelie_results.tsv benchmark_file_conversion_data.tsv 1> amelie.log 2> amelie.err: [evidence span withheld — anchor status: not_found]
