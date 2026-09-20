---
name: causal-gene-rank-threshold-counting
description: Use when you have per-case ranked gene lists from multiple phenotype-prioritization
  tools on a common benchmark and need to count causal genes uniquely detected by
  one tool within top-N cutoffs, e.g. N=1..20, to characterize stability and complementarity.
when_to_use_negative:
- The input is a single tool's output without matched ranked lists for all eight tools,
  since unique-hit counting requires the full comparator set on the same cases
- You need raw prioritization performance (recall/rank distributions) rather than
  between-tool complementarity — Figure 1/2-style rank analyses cover that instead
- The benchmark relies on PhenoTips as a comparator and no local installation is possible,
  since PhenoTips no longer offers a stand-alone download and requires a paid cloud
  subscription (deemed obsolete in the benchmarking README)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VIBE
  role: Phenotype-based gene prioritizer whose ranked outputs (and those of the seven
    comparators) are the subject of the unique-hit threshold analysis; run as vibe-with-dependencies.jar
    with HPO terms
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Benchmark runners (e.g. VibeBenchmarkRunner, ExomiserBenchmarkRunner) and
    BenchmarkFileGeneSymbolToIdConverter generate and harmonize the per-tool ranked
    result files that feed the top-N counting analysis
  repo: https://github.com/molgenis/vibe-suppl
- name: R
  role: Visualization of count-versus-threshold curves (PaperPlots.R) for the unique-hit
    analysis across cutoffs
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:4adb66dfa5b4896a41af13f9d582c865a7efe49280b1f3bac47945a501293f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Python
  - R
  - VIBE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# causal-gene-rank-threshold-counting

## Summary

Skill for quantifying how many benchmark causal genes a phenotype-based gene prioritization tool (e.g. VIBE) uniquely detects within top-N ranked results, swept across thresholds N=1..20, as used to analyze VIBE's performance against Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE and GADO on 305 patient cases.

## When to use

Use when you have per-case ranked gene lists from multiple phenotype-prioritization tools on a common benchmark and need to count causal genes uniquely detected by one tool within top-N cutoffs, e.g. N=1..20, to characterize stability and complementarity.

## When NOT to use

- The input is a single tool's output without matched ranked lists for all eight tools, since unique-hit counting requires the full comparator set on the same cases
- You need raw prioritization performance (recall/rank distributions) rather than between-tool complementarity — Figure 1/2-style rank analyses cover that instead
- The benchmark relies on PhenoTips as a comparator and no local installation is possible, since PhenoTips no longer offers a stand-alone download and requires a paid cloud subscription (deemed obsolete in the benchmarking README)

## Inputs

- Per-case ranked gene lists (one per tool: VIBE, Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO) from the 305-patient-case / 308-causal-gene benchmark
- benchmark_data.tsv mapping case IDs to comma-separated HPO phenotype names compatible with HPO release 2018-03-08
- benchmark_file_conversion_data.tsv with HGNC gene symbols, previous symbols and NCBI gene IDs (from genenames.org) for harmonizing gene identifiers

## Outputs

- Tab-delimited table of uniquely detected causal gene counts per tool per threshold N=1..20
- Count-versus-threshold curve plot (Figure 4-style) generated with R
- Validation report confirming total recall of 308 causal genes at rank 20

## How to apply

Load the per-case ranked gene lists produced by the benchmark in which each of the 308 causal genes from 305 patient cases was spiked into 19 pseudorandom CGD genes, with each rank stabilized as the median of 25 permutations. In Python, for each tool and each threshold N=1..20, flag whether the causal gene appears within the top N, then count causal genes detected within top N by the focal tool (e.g. VIBE) but by none of the seven comparators (Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO). Summarize the unique-hit counts for each N in a tab-delimited table and visualize the count-versus-threshold curve in R, mirroring Figure 4. Validate that at threshold N=20 the analysis reproduces the paper's total recall of 308 causal genes for all tools, and check that the focal tool's unique-hit counts fluctuate little across N=1..20, which the paper presents as stable performance (VIBE had 6 unique hits in the top 20 vs GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, Phenomizer 15, AMELIE 17).

## Related tools

- **VIBE** (Phenotype-based gene prioritizer whose ranked outputs (and those of the seven comparators) are the subject of the unique-hit threshold analysis; run as vibe-with-dependencies.jar with HPO terms) — https://github.com/molgenis/vibe
- **Python** (Benchmark runners (e.g. VibeBenchmarkRunner, ExomiserBenchmarkRunner) and BenchmarkFileGeneSymbolToIdConverter generate and harmonize the per-tool ranked result files that feed the top-N counting analysis) — https://github.com/molgenis/vibe-suppl
- **R** (Visualization of count-versus-threshold curves (PaperPlots.R) for the unique-hit analysis across cutoffs) — https://github.com/molgenis/vibe-suppl

## Examples

```
python3 BenchmarkFileGeneSymbolToIdConverter.py amelie_results.tsv benchmark_file_conversion_data.tsv 1> amelie.log 2> amelie.err
```

## Evaluation signals

- At threshold N=20 the analysis reaches the paper's reported total recall of 308 causal genes for all tools
- VIBE's unique-hit count at N=20 equals 6, with comparator counts GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, Phenomizer 15, AMELIE 17
- VIBE's unique-hit counts show no large fluctuation across N=1..20, consistent with the reported stable performance
- All gene identifiers are consistently converted to NCBI gene IDs via BenchmarkFileGeneSymbolToIdConverter before comparing tools
- Per-case causal gene ranks are medians over 25 permutations, so recomputing them should reproduce stable ranks without outliers

## Limitations

- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions, which may affect unique-hit counts
- Unique-hit comparisons at higher thresholds conflate complementarity with total recall differences, since all tools only arrive at their total recall of 308 causal genes at rank 20

## Evidence

- [methods] Simulate clinical exome interpretation with 20 candidate genes (causal gene spiked into 19 pseudorandom CGD genes): "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [methods] Ranks stabilized as median of 25 permutations: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] Total recall at rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] Benchmark provenance: [evidence span withheld — anchor status: not_found]
