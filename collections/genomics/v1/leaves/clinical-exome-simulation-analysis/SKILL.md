---
name: clinical-exome-simulation-analysis
description: Use when benchmarking phenotype-based gene prioritization tools (e.g.
  VIBE vs Phenomizer, AMELIE, GADO) on patient-case HPO terms and you need top-N causal-gene
  recall and unique-hit counts at a 20-gene clinical-exome panel size.
when_to_use_negative:
- When evaluating tools on real diagnostic pipelines with structural variation, CGH,
  Sanger sequencing, RNA-sequencing or metabolomics inputs — the benchmark only applies
  to phenotype-gene matching for SNV/indel-style exome candidate lists
- When a tool requires a paid cloud subscription or stand-alone installation no longer
  available (e.g. PhenoTips as of January 2020) — the benchmark arm is obsolete and
  cannot be reproduced
- When the goal is to demonstrate individual strengths of a single tool rather than
  comparative top-20 recall — the authors note the benchmark 'does not demonstrate
  individual strengths of the tested tools'
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3325
tools:
- name: VIBE
  role: Phenotype-based gene prioritizer under evaluation; produces ranked gene lists
    from HPO terms via DisGeNET GDAs
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Runs benchmark runner/generator scripts, converts HGNC symbols to NCBI gene
    IDs, computes per-threshold top-N unique-hit counts with pandas
- name: R
  role: Visualizes unique-hit count-versus-threshold curves (PaperPlots.R, Figure
    4)
  repo: https://github.com/molgenis/vibe-suppl
- name: AMELIE
  role: Comparator tool ranked gene lists used to determine whether VIBE's top-N hits
    are unique
- name: Phenomizer
  role: Comparator tool; top scorer for unique top-20 hits (15)
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d2ce78736a2d2ab6f3a88e8601df447890b847c8fb97e9b10fdf72a3e36f8547
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# clinical-exome-simulation-analysis

## Summary

Simulates clinical exome interpretation by spiking each benchmark causal gene into 19 pseudorandom CGD genes, ranking candidate gene lists with VIBE and seven comparator tools, and scoring top-20 causal-gene recall and uniquely detected causal genes per tool.

## When to use

Use when benchmarking phenotype-based gene prioritization tools (e.g. VIBE vs Phenomizer, AMELIE, GADO) on patient-case HPO terms and you need top-N causal-gene recall and unique-hit counts at a 20-gene clinical-exome panel size.

## When NOT to use

- When evaluating tools on real diagnostic pipelines with structural variation, CGH, Sanger sequencing, RNA-sequencing or metabolomics inputs — the benchmark only applies to phenotype-gene matching for SNV/indel-style exome candidate lists
- When a tool requires a paid cloud subscription or stand-alone installation no longer available (e.g. PhenoTips as of January 2020) — the benchmark arm is obsolete and cannot be reproduced
- When the goal is to demonstrate individual strengths of a single tool rather than comparative top-20 recall — the authors note the benchmark 'does not demonstrate individual strengths of the tested tools'

## Inputs

- Per-patient-case HPO codes matched to HPO release v2018-03-08 (benchmark_data.tsv with ID and comma-separated phenotype names)
- CGD (Clinical Genomic Database, accessed 4 February 2020) gene list as candidate gene pool (n=3986 CGD NCBI identifiers)
- Per-case ranked gene lists for VIBE and Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, GADO (converted to NCBI gene IDs via genenames.org mapping file)
- hp.obo Human Phenotype Ontology (release 2018-03-08)
- benchmark_file_conversion_data.tsv (HGNC symbols with previous symbols and NCBI gene IDs)

## Outputs

- Per-rank counts of causal genes ranked first through rank 20 per tool
- Cumulative top-20 recall per tool (expected to reach 308 causal genes at rank 20)
- Unique-hit counts per tool at thresholds N=1..20 (reported: GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, AMELIE 17)
- vibe_top20_recall_unique_hits.csv with per-rank counts, top-20 recall, and unique-hit counts
- Rank-stabilized causal gene ranks (median of 25 permutations) per case per tool

## How to apply

For each of the 308 causal genes from the 305 patient cases, select 19 pseudorandom genes from the CGD (Clinical Genomic Database) to build 308 simulated 20-gene panels, then supply the HPO codes (matched to HPO release v2018-03-08) to VIBE and the seven comparator tools. For genes absent from a tool's output, assign a random rank after the returned genes (in practice the causal gene falls between rank 16 and 20) and stabilize each rank as the median of 25 permutations to compensate for outliers. Using Python/pandas, count per rank position how many causal genes are ranked first through rank 20 and derive cumulative top-20 recall, which should reach the full 308 causal genes at rank 20. For unique-hit analysis, for each threshold N=1..20 flag cases where the causal gene is within top N for VIBE but for none of the seven other tools, and visualize the count-versus-threshold curve in R (Figure 4). Write per-rank counts, top-20 recall, and unique-hit counts to CSV for comparison against the paper's reported values (e.g. VIBE unique top-20 hits = 6).

## Related tools

- **VIBE** (Phenotype-based gene prioritizer under evaluation; produces ranked gene lists from HPO terms via DisGeNET GDAs) — https://github.com/molgenis/vibe
- **Python** (Runs benchmark runner/generator scripts, converts HGNC symbols to NCBI gene IDs, computes per-threshold top-N unique-hit counts with pandas)
- **R** (Visualizes unique-hit count-versus-threshold curves (PaperPlots.R, Figure 4)) — https://github.com/molgenis/vibe-suppl
- **AMELIE** (Comparator tool ranked gene lists used to determine whether VIBE's top-N hits are unique)
- **Phenomizer** (Comparator tool; top scorer for unique top-20 hits (15))

## Examples

```
python3 VibeBenchmarkParallelBashScriptsGenerator.py hp.obo benchmark_data.tsv ./ 50 && python3 VibeBenchmarkFileGenerator.py results/ vibe_results.tsv none
```

## Evaluation signals

- Cumulative recall reaches the total of 308 causal genes at rank 20 for every tool
- VIBE's unique top-20 hit count equals the reported value of 6 (ranked below Phenomizer's 15 and AMELIE's 17)
- Comparator unique-hit counts match the reported ordering: GADO 1, PubCaseFinder 1, Phenotips 2, hiPHIVE 3, PhenIX 5, VIBE 6, Phenomizer 15, AMELIE 17
- VIBE's unique-hit counts show no large fluctuation across N = 1..20, consistent with the paper's reported stable performance
- Per-case causal gene ranks are stabilized as the median of 25 permutations, with randomly assigned ranks for genes absent from output falling between rank 16 and 20

## Limitations

- Ranks for causal genes absent from a tool's output must be randomly assigned after the returned genes, introducing stochasticity compensated only by median-of-25-permutation stabilization
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions, unresolved at time of publication
- PhenoTips benchmark arm is obsolete because Phenotips no longer offers a stand-alone downloadable solution as of January 2020

## Evidence

- [methods] Simulate clinical exome interpretation with 20 candidate genes (causal gene spiked into 19 pseudorandom CGD genes): "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [methods] Stabilize ranks as median of 25 permutations: "To compensate for outliers, we stabilized the rank as the median of 25 permutations"
- [results] Total recall at rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [methods] Comparator tool set: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark repo and plotting scripts: [evidence span withheld — anchor status: not_found]
- [readme] HPO release pinned for benchmark: [evidence span withheld — anchor status: not_found]
