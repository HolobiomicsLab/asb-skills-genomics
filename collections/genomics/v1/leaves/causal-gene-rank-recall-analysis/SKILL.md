---
name: causal-gene-rank-recall-analysis
description: Use when comparing phenotype-based gene prioritization tools on a patient-case
  benchmark and you need per-rank causal-gene counts, top-20 recall, or counts of
  causal genes uniquely detected in the top 20 by one tool.
when_to_use_negative:
- You need tool-specific strengths near the bottom of the rank distribution — the
  paper notes VIBE lacks exceptionally well prioritized genes there and the benchmark
  does not demonstrate individual strengths of the tested tools.
- You want to benchmark PhenoTips — as of January 2020 it no longer offers a stand-alone
  downloadable solution, so that benchmark is deemed obsolete in the vibe-suppl README.
- Your aim is variant-level candidate filtering (allele frequency, inheritance mode,
  pathogenicity) rather than gene-level rank/recall evaluation of phenotype prioritizers.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0089
tools:
- name: VIBE
  role: Phenotype-based gene prioritizer whose ranked output lists are evaluated for
    causal-gene rank, top-20 recall, and unique top-20 hits
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Used with pandas to load ranked lists, apply random-rank assignment and 25-permutation
    median stabilization, and compute per-rank counts and unique hits
- name: R
  role: Used for benchmarking results processing (PaperPlots.R in vibe-suppl)
  repo: https://github.com/molgenis/vibe-suppl
- name: BenchmarkFileGeneSymbolToIdConverter.py
  role: Converts HGNC gene symbols in each tool's results to NCBI gene IDs so ranks
    are comparable across tools
  repo: https://github.com/molgenis/vibe-suppl
- name: Phenomizer
  role: Benchmarked comparison tool whose top-20 detections define the unique-hit
    counts against VIBE (15 unique hits)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:06ffa5270da915a4831aca4d2fd619df550fc1b7130610c876787991960cf8f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - BenchmarkFileGeneSymbolToIdConverter.py
  - Phenomizer
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# causal-gene-rank-recall-analysis

## Summary

Reproduce the VIBE benchmark's causal-gene rank analysis: per-rank counts, cumulative top-20 recall (reaching 308 causal genes at rank 20), and unique top-20 hits (6 for VIBE), by stabilizing each case's causal-gene rank as the median of 25 permutations and counting unique detections across benchmarked tools.

## When to use

Use when comparing phenotype-based gene prioritization tools on a patient-case benchmark and you need per-rank causal-gene counts, top-20 recall, or counts of causal genes uniquely detected in the top 20 by one tool.

## When NOT to use

- You need tool-specific strengths near the bottom of the rank distribution — the paper notes VIBE lacks exceptionally well prioritized genes there and the benchmark does not demonstrate individual strengths of the tested tools.
- You want to benchmark PhenoTips — as of January 2020 it no longer offers a stand-alone downloadable solution, so that benchmark is deemed obsolete in the vibe-suppl README.
- Your aim is variant-level candidate filtering (allele frequency, inheritance mode, pathogenicity) rather than gene-level rank/recall evaluation of phenotype prioritizers.

## Inputs

- Per-case ranked gene lists from VIBE and the seven other benchmarked prioritization tools (e.g., vibe_results.tsv, amelie_results.tsv, phenomizer_results.tsv, converted to NCBI gene IDs)
- benchmark_data.tsv with case ID and comma-separated HPO phenotype names compatible with HPO release 2018-03-08
- benchmark_file_conversion_data.tsv mapping HGNC symbols (including previous symbols) to NCBI gene IDs from genenames.org

## Outputs

- vibe_top20_recall_unique_hits.csv with per-rank causal-gene counts, cumulative top-20 recall, and unique top-20 hit counts
- Verified unique top-20 hit count per tool (VIBE: 6; Phenomizer: 15; AMELIE: 17)

## How to apply

Load the per-case ranked gene lists (for VIBE: 308 simulated clinical exome panels, each with the causal gene spiked into 19 pseudorandom CGD genes drawn from the n=3986 CGD NCBI identifier pool) using Python/pandas. Apply the paper's rule for causal genes absent from a tool's output: assign a random rank positioned after the returned genes, since in practice the causal gene falls between rank 16 and 20 in that scenario. Stabilize each case's causal-gene rank as the median of 25 permutations to compensate for outliers introduced by the random-rank assignment. Count, per rank position up to 20, how many causal genes are ranked first, second, third, and so on, and derive the cumulative top-20 recall, which should reach the full 308 causal genes at rank 20. Finally, compute the number of causal genes within the top 20 detected by one tool (e.g., VIBE) that no other benchmarked tool (Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO) detects within its top 20, and write per-rank counts, recall, and unique-hit counts to a CSV for comparison against reported figures.

## Related tools

- **VIBE** (Phenotype-based gene prioritizer whose ranked output lists are evaluated for causal-gene rank, top-20 recall, and unique top-20 hits) — https://github.com/molgenis/vibe
- **Python** (Used with pandas to load ranked lists, apply random-rank assignment and 25-permutation median stabilization, and compute per-rank counts and unique hits)
- **R** (Used for benchmarking results processing (PaperPlots.R in vibe-suppl)) — https://github.com/molgenis/vibe-suppl
- **BenchmarkFileGeneSymbolToIdConverter.py** (Converts HGNC gene symbols in each tool's results to NCBI gene IDs so ranks are comparable across tools) — https://github.com/molgenis/vibe-suppl
- **Phenomizer** (Benchmarked comparison tool whose top-20 detections define the unique-hit counts against VIBE (15 unique hits))

## Examples

```
python3 VibeBenchmarkFileGenerator.py results/ vibe_results.tsv none
```

## Evaluation signals

- Cumulative top-20 recall reaches the full 308 causal genes at rank 20 (matching the paper: 'At rank 20, all tools arrive at their total recall of 308 causal genes')
- VIBE's unique top-20 hit count equals 6, exceeded only by Phenomizer (15) and AMELIE (17)
- Every tool result file has been converted to NCBI gene identifiers via BenchmarkFileGeneSymbolToIdConverter.py before ranking comparison
- Causal-gene ranks are stabilized as the median of 25 permutations, so repeated runs converge rather than flipping due to random-rank assignment
- Missed causal genes (absent from tool output) are counted and reported per tool, as in Figure 1's labels

## Limitations

- Genes absent from VIBE output require a random rank after the returned genes, introducing variance that must be absorbed by the 25-permutation median
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions, which the authors aim to resolve in upcoming versions
- External services such as AMELIE cannot always be used since privacy and availability are not guaranteed

## Evidence

- [methods] simulated clinical exome panels: "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [methods] rank stabilization rule: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] total recall at rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [abstract] benchmark design: "We evaluate VIBE in a benchmark based on 305 patient cases alongside seven other tools."
- [results] unique-hit comparison: [evidence span withheld — anchor status: not_found]
- [readme] symbol-to-ID conversion: [evidence span withheld — anchor status: not_found]
