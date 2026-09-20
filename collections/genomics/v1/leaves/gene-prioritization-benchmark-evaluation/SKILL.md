---
name: gene-prioritization-benchmark-evaluation
description: Use when reproducing or comparing phenotype-based gene prioritizer benchmark
  results, e.g. counting causal-gene ranks, top-20 recall, or unique top-N hit counts
  across VIBE, Phenomizer, AMELIE, GADO, PhenIX, hiPHIVE, PhenoTips and PubCaseFinder.
when_to_use_negative:
- The goal is to run VIBE on new patient samples for diagnostics rather than evaluate
  benchmark performance against published rank metrics
- Only a single tool's output is available with no comparator results, so unique-hit
  counts across tools cannot be computed
- The case phenotype data are not compatible with HPO release 2018-03-08, so ranks
  would not be comparable to the paper's benchmark
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3325
tools:
- name: VIBE
  role: Prioritizes disease genes from HPO codes; its per-case ranked gene lists are
    the primary benchmark output evaluated for top-20 recall and unique hits
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Loads ranked lists, assigns random ranks to absent causal genes, computes
    median-of-25-permutation rank stabilization, and counts top-N unique hits per
    threshold
- name: R
  role: Visualizes the unique-hit count-versus-threshold curve for N = 1..20 (PaperPlots.R
    in the benchmark repository)
  repo: https://github.com/molgenis/vibe-suppl
- name: Phenomizer
  role: Comparator phenotype-prioritization tool; uniquely detects 15 causal genes
    within the top 20
- name: AMELIE
  role: Comparator phenotype-prioritization tool; uniquely detects 17 causal genes
    within the top 20
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d0afad71a6a56c7aa47b5d0d0cc48c7c60184d8cccfecd2de3ab1bf952461d23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - AMELIE
  - Phenomizer
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-prioritization-benchmark-evaluation

## Summary

Evaluates phenotype-based gene prioritization tools (VIBE and seven comparators) on a 305-patient-case benchmark in which each of 308 causal genes is spiked into 19 pseudorandom CGD genes, counting per-rank causal-gene recall and top-N unique hits unique to a single tool.

## When to use

Use when reproducing or comparing phenotype-based gene prioritizer benchmark results, e.g. counting causal-gene ranks, top-20 recall, or unique top-N hit counts across VIBE, Phenomizer, AMELIE, GADO, PhenIX, hiPHIVE, PhenoTips and PubCaseFinder.

## When NOT to use

- The goal is to run VIBE on new patient samples for diagnostics rather than evaluate benchmark performance against published rank metrics
- Only a single tool's output is available with no comparator results, so unique-hit counts across tools cannot be computed
- The case phenotype data are not compatible with HPO release 2018-03-08, so ranks would not be comparable to the paper's benchmark

## Inputs

- Per-case ranked gene lists (NCBI gene IDs) for VIBE and seven comparator tools (Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO), e.g. vibe_results.tsv, amelie_results.tsv, phenomizer_results.tsv
- benchmark_data.tsv with case IDs and comma-separated phenotype names (compatible with HPO release 2018-03-08)
- benchmark_file_conversion_data.tsv mapping HGNC gene symbols to NCBI gene IDs
- hp.obo (Human Phenotype Ontology)

## Outputs

- Per-rank counts of causal genes ranked 1st, 2nd, ... up to rank 20 and cumulative top-20 recall per tool
- Tab-delimited table of unique-hit counts (genes in top N by one tool only) per threshold N = 1..20, e.g. vibe_top20_recall_unique_hits.csv
- R visualization of unique-hit count versus threshold (Figure 4-style curve)

## How to apply

Load the per-case ranked gene lists (NCBI gene IDs) for VIBE and the seven comparator tools from the 308 simulated panels, each with the causal gene spiked into 19 pseudorandom CGD genes. For genes absent from a tool's output, assign a random rank after the returned genes, then stabilize each case's causal-gene rank as the median of 25 permutations to compensate for outliers. For each threshold N = 1..20, determine whether the causal gene is ranked within the top N, and count causal genes detected within top N by one tool but by no other tool (unique hits); VIBE's unique top-20 count should be 6, exceeded only by Phenomizer (15) and AMELIE (17). Validation requires cumulative recall to reach the full 308 causal genes at rank 20 for all tools, and VIBE's unique-hit counts should show no large fluctuation across N = 1..20.

## Related tools

- **VIBE** (Prioritizes disease genes from HPO codes; its per-case ranked gene lists are the primary benchmark output evaluated for top-20 recall and unique hits) — https://github.com/molgenis/vibe
- **Python** (Loads ranked lists, assigns random ranks to absent causal genes, computes median-of-25-permutation rank stabilization, and counts top-N unique hits per threshold)
- **R** (Visualizes the unique-hit count-versus-threshold curve for N = 1..20 (PaperPlots.R in the benchmark repository)) — https://github.com/molgenis/vibe-suppl
- **Phenomizer** (Comparator phenotype-prioritization tool; uniquely detects 15 causal genes within the top 20)
- **AMELIE** (Comparator phenotype-prioritization tool; uniquely detects 17 causal genes within the top 20)

## Examples

```
python3 VibeBenchmarkFileGenerator.py results/ vibe_results.tsv none
```

## Evaluation signals

- Cumulative count reaches total recall of 308 causal genes at rank 20 for all tools
- VIBE's unique top-20 hit count equals 6, exceeded only by Phenomizer (15) and AMELIE (17)
- Unique-hit counts across comparators at N=20 match GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, AMELIE 17
- Median-of-25-permutation rank stabilization removes outliers introduced by random-rank assignment for absent causal genes
- VIBE's unique-hit counts show no large fluctuation across thresholds N = 1..20, consistent with the reported stable performance

## Limitations

- Genes absent from a tool's output require a random rank assignment after the returned genes, introducing variance handled only by median-of-25-permutation stabilization
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions, and lacks exceptionally well prioritized genes near the bottom of the rank distribution
- External services such as AMELIE cannot always be used since privacy and availability are not guaranteed; PhenoTips no longer offers a stand-alone downloadable solution

## Evidence

- [methods] Simulate clinical exome interpretation with 20 candidate genes (causal gene spiked into 19 pseudorandom CGD genes): "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [methods] Stabilize ranks as median of 25 permutations: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] Total recall of 308 causal genes at rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [abstract] VIBE benchmarked on 305 patient cases alongside seven other tools: "We evaluate VIBE in a benchmark based on 305 patient cases alongside seven other tools."
- [readme] Benchmark repository location: [evidence span withheld — anchor status: not_found]
