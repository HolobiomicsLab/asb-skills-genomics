---
name: tool-complementarity-unique-hit-analysis
description: Use when comparing ranked gene-prioritization outputs (e.g. VIBE vs Phenomizer,
  PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO) to determine which causal
  genes each tool uniquely detects and at which top-N cutoffs.
when_to_use_negative:
- When the goal is overall recall or mean rank performance rather than tool-specific
  complementarity — use the full rank-distribution analysis (Figure 1) instead
- When tool outputs are not comparable ranked gene lists harmonized to a common identifier
  space (e.g. HGNC symbols not converted to NCBI gene IDs), as unique-hit counting
  requires cross-tool gene identity
- When the benchmark has no ground-truth causal gene per case, since 'unique hit'
  is defined strictly against known causal genes
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0085
tools:
- name: VIBE
  role: Primary prioritization tool whose unique-hit counts across top-N thresholds
    are analyzed
  repo: https://github.com/molgenis/vibe
- name: vibe-suppl
  role: Supplemental benchmarking scripts (benchmark runners and file generators)
    producing the per-tool ranked results used in the unique-hit analysis; PaperPlots.R
    generates the benchmark figures
  repo: https://github.com/molgenis/vibe-suppl
- name: Python
  role: Within-top-N hit testing and unique-hit counting across thresholds N = 1..20
    (e.g. BenchmarkFileGeneSymbolToIdConverter.py, *BenchmarkRunner.py scripts)
  repo: https://github.com/molgenis/vibe-suppl
- name: R
  role: Visualization of unique-hit counts versus threshold (Figure 4, PaperPlots.R)
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:243fe9bc8fcdd1382d7f5066a627d1559b0d5ca493e638933486325202b890b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Python
  - R
  - VIBE
  - vibe-suppl
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tool-complementarity-unique-hit-analysis

## Summary

Quantifies benchmark complementarity among phenotype-based gene prioritization tools by counting, per rank threshold N, causal genes recovered within the top N by one tool but by none of the comparator tools, revealing which hits each tool uniquely contributes.

## When to use

Use when comparing ranked gene-prioritization outputs (e.g. VIBE vs Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO) to determine which causal genes each tool uniquely detects and at which top-N cutoffs.

## When NOT to use

- When the goal is overall recall or mean rank performance rather than tool-specific complementarity — use the full rank-distribution analysis (Figure 1) instead
- When tool outputs are not comparable ranked gene lists harmonized to a common identifier space (e.g. HGNC symbols not converted to NCBI gene IDs), as unique-hit counting requires cross-tool gene identity
- When the benchmark has no ground-truth causal gene per case, since 'unique hit' is defined strictly against known causal genes

## Inputs

- Per-case ranked gene output lists (e.g. results.tsv) for VIBE and seven comparator tools (Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO), gene identifiers harmonized to NCBI gene IDs
- Benchmark case definition mapping each of the 305 patient cases / 308 causal genes to its HPO terms and the spiked-in 20-gene candidate panel (causal gene + 19 pseudorandom CGD genes)
- Known causal-gene label per case for within-top-N hit testing

## Outputs

- Tab-delimited table of unique-hit counts per tool per threshold N = 1..20
- Count-versus-threshold curve (R plot, analogous to Figure 4) showing unique hits for all tools across cutoffs
- Validation report confirming total recall of 308 causal genes at rank 20

## How to apply

Load per-case ranked gene lists for all eight tools from the 305-patient-case benchmark in which each of the 308 causal genes was spiked into 19 pseudorandom CGD genes, with ranks stabilized as the median of 25 permutations. For each case and each threshold N = 1..20, test whether the causal gene falls within the top N for each tool using Python. Per threshold, count causal genes ranked within top N by the tool of interest (e.g. VIBE) but by none of the seven comparators. Summarize unique-hit counts across N = 1..20 in a tab-delimited table and plot the count-versus-threshold curve in R (as in Figure 4, where VIBE had 6 unique hits at top 20 vs GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, Phenomizer 15, AMELIE 17). Validate that at N = 20 the analysis reproduces the paper's total recall of 308 causal genes, and check that unique-hit counts fluctuate little across the N = 1..20 range as a stability signal.

## Related tools

- **VIBE** (Primary prioritization tool whose unique-hit counts across top-N thresholds are analyzed) — https://github.com/molgenis/vibe
- **vibe-suppl** (Supplemental benchmarking scripts (benchmark runners and file generators) producing the per-tool ranked results used in the unique-hit analysis; PaperPlots.R generates the benchmark figures) — https://github.com/molgenis/vibe-suppl
- **Python** (Within-top-N hit testing and unique-hit counting across thresholds N = 1..20 (e.g. BenchmarkFileGeneSymbolToIdConverter.py, *BenchmarkRunner.py scripts)) — https://github.com/molgenis/vibe-suppl
- **R** (Visualization of unique-hit counts versus threshold (Figure 4, PaperPlots.R)) — https://github.com/molgenis/vibe-suppl

## Examples

```
python3 BenchmarkFileGeneSymbolToIdConverter.py vibe_results.tsv benchmark_file_conversion_data.tsv 1> vibe.log 2> vibe.err
```

## Evaluation signals

- At threshold N = 20 the unique-hit accounting reaches the paper's total recall of 308 causal genes across all tools
- Top-20 unique-hit counts match the reported values: GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, AMELIE 17
- VIBE's unique-hit counts show no large fluctuation across N = 1..20, consistent with reported stable performance
- Every counted unique hit is verifiable: the causal gene is present within top N for exactly one tool and absent from the top-N output of the other seven
- All per-case gene lists are converted to a common identifier space (NCBI gene IDs via BenchmarkFileGeneSymbolToIdConverter.py) before comparison

## Limitations

- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE was noted to lack exceptionally well prioritized genes near the bottom of the rank distribution and to show a bias with over-represented genes (MECP2, KMT2D) at high positions
- External services such as AMELIE may offer valuable information but cannot always be used since privacy and availability are not guaranteed
- Some benchmark components may be obsolete — as of January 2020 PhenoTips no longer offers a stand-alone downloadable solution and the benchmark path was deemed obsolete in the supplementary README

## Evidence

- [results] Within the top 20 results, the unique-hit counts were GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, and AMELIE 17: [evidence span withheld — anchor status: not_found]
- [results] At rank 20, all tools arrive at their total recall of 308 causal genes.: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [methods] For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] Hierarchical clustering shows a degree of dissimilarity between all of the tested tools.: "Causal genes that were not observed in the tool output are shown in black. Hierarchical clustering shows a degree of dissimilarity between all of the tested tools."
- [readme] python3 BenchmarkFileGeneSymbolToIdConverter.py amelie_results.tsv benchmark_file_conversion_data.tsv 1> amelie.log 2> amelie.err: [evidence span withheld — anchor status: not_found]
