---
name: benchmark-result-visualization
description: Use when ranked gene lists from a multi-tool benchmark (e.g. 305 patient
  cases, 308 causal genes spiked into 19 CGD genes) must be compared across tools
  and top-N thresholds via plots.
when_to_use_negative:
- Only a single tool's output exists and no cross-tool comparison or unique-hit analysis
  is intended
- Results have not been converted to a common gene identifier space (HGNC symbols
  vs NCBI gene IDs), which would corrupt rank matching across tools
- Benchmark cases lack a known causal gene to rank, so rank-based figures cannot be
  constructed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Renders the benchmark plots, including the count-versus-threshold curve for
    VIBE's unique hits; PaperPlots.R in vibe-suppl regenerates the paper figures
  repo: https://github.com/molgenis/vibe-suppl
- name: Python
  role: Computes per-threshold top-N and unique-hit counts from the per-case ranked
    gene lists before plotting
  repo: https://github.com/molgenis/vibe-suppl
- name: VIBE
  role: Benchmarked tool whose ranks are visualized against the seven comparators;
    runs as a Java 8 command-line jar producing results.tsv
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:9f72063b05c0ef93936c5b74d58c41d600667fae40bd8093184efa4716bbceb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe-suppl
  tools:
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-result-visualization

## Summary

Visualizes phenotype-based gene-prioritization benchmark results by plotting causal-gene rank distributions, per-case rank heat maps, and unique-hit counts across top-N cutoffs (N = 1..20) for VIBE versus seven comparator tools, reproducing the paper's Figures 1, 2 and 4.

## When to use

Use when ranked gene lists from a multi-tool benchmark (e.g. 305 patient cases, 308 causal genes spiked into 19 CGD genes) must be compared across tools and top-N thresholds via plots.

## When NOT to use

- Only a single tool's output exists and no cross-tool comparison or unique-hit analysis is intended
- Results have not been converted to a common gene identifier space (HGNC symbols vs NCBI gene IDs), which would corrupt rank matching across tools
- Benchmark cases lack a known causal gene to rank, so rank-based figures cannot be constructed

## Inputs

- Per-case ranked gene lists per tool (tool results TSVs, e.g. vibe_results.tsv, phenomizer_results.tsv, amelie_results.tsv, converted to NCBI gene IDs)
- benchmark_data.tsv (case ID plus comma-separated HPO phenotype names, compatible with HPO release 2018-03-08)
- List of 308 causal genes from the 305 patient-case benchmark

## Outputs

- Rank-distribution plot (number of returned genes vs causal-gene rank, missed genes in labels; Figure 1)
- Per-case causal-gene rank heat map with hierarchical clustering of tools (Figure 2)
- Unique-hit count-versus-threshold curve for N = 1..20 (Figure 4)
- Tab-delimited table of unique-hit counts per threshold

## How to apply

Load the per-case ranked gene lists (benchmark results as TSV with HGNC symbols converted to NCBI gene IDs via BenchmarkFileGeneSymbolToIdConverter.py) for all eight tools. In Python, for each case and each tool, determine for every threshold N = 1..20 whether the causal gene falls within the top N, then count genes detected uniquely by VIBE (not by Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE or GADO). Visualize three views: (1) number of returned genes vs causal-gene rank with missed-gene counts in labels (Figure 1), (2) a heat map of causal-gene rank per patient case per tool with unobserved causal genes shown in black plus hierarchical clustering to expose tool dissimilarity (Figure 2), and (3) the unique-hit count-versus-threshold curve across N = 1..20 (Figure 4). Use R, e.g. the PaperPlots.R script from the vibe-suppl repository, whose required files are archived at Zenodo record 3662470.

## Related tools

- **R** (Renders the benchmark plots, including the count-versus-threshold curve for VIBE's unique hits; PaperPlots.R in vibe-suppl regenerates the paper figures) — https://github.com/molgenis/vibe-suppl
- **Python** (Computes per-threshold top-N and unique-hit counts from the per-case ranked gene lists before plotting) — https://github.com/molgenis/vibe-suppl
- **VIBE** (Benchmarked tool whose ranks are visualized against the seven comparators; runs as a Java 8 command-line jar producing results.tsv) — https://github.com/molgenis/vibe

## Examples

```
Rscript benchmarking_results_processing/PaperPlots.R
```

## Evaluation signals

- At threshold N = 20 the analysis reaches the paper's total recall of 308 causal genes for every tool
- Unique-hit counts at top-20 reproduce the reported values: GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, AMELIE 17
- VIBE's unique-hit counts show no large fluctuation across N = 1..20, consistent with the reported stable performance
- Heat map shows black cells exactly where a tool's output omitted the causal gene, and hierarchical clustering separates dissimilar tools
- Ranks used for plotting are the medians of 25 permutations per case, not single-run ranks

## Limitations

- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- PhenoTips benchmarking is obsolete since it no longer offers a stand-alone downloadable solution, so its visualized results cannot be regenerated with the original setup
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution and shows a bias with over-represented genes (MECP2, KMT2D) at high positions

## Evidence

- [results] Figure 1 rank plot: "Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list."
- [results] Figure 2 heat map: "Causal genes that were not observed in the tool output are shown in black. Hierarchical clustering shows a degree of dissimilarity between all of the tested tools."
- [results] At rank 20 all tools reach total recall: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] PaperPlots.R availability: [evidence span withheld — anchor status: not_found]
- [other] R visualization of unique-hit curve: [evidence span withheld — anchor status: not_found]
