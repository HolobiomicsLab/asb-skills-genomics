---
name: prioritization-bias-analysis
description: Use when benchmarking a phenotype-based gene prioritizer and you suspect
  certain genes recur at high positions across many cases; count top-N gene occurrences
  per ranked case output to expose such bias.
when_to_use_negative:
- The tool output is unranked or already collapsed into a single gene set rather than
  per-case ranked gene lists
- The goal is overall ranking accuracy (e.g., rank-of-causal-gene distributions, recall
  at rank 20) rather than detecting recurring over-represented genes
- Analysis targets structural variants or variant-level filtering (allele frequency,
  pathogenicity) rather than gene-prioritization output
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3325
tools:
- name: VIBE
  role: Source of the per-case ranked gene prioritization output (tab-delimited, sorted
    by highest GDA score) that is tallied for over-represented genes
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Parsing each case's ranked gene list with pandas, extracting top-N identifiers,
    and tallying gene occurrences
  repo: https://github.com/molgenis/vibe-suppl
- name: R
  role: Benchmark result processing and plotting of the per-case gene-rank data
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:14a37a74ff5a52b021b770e922363363f7e1227f1dff070200d2bb06cd8290dc
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# prioritization-bias-analysis

## Summary

Quantifies over-representation of specific genes in the top-ranked positions of a gene prioritization tool's per-case output lists, as done for MECP2 (NCBI 4204) and KMT2D (NCBI 8085) in the VIBE benchmark of 305 rare-disease patient cases.

## When to use

Use when benchmarking a phenotype-based gene prioritizer and you suspect certain genes recur at high positions across many cases; count top-N gene occurrences per ranked case output to expose such bias.

## When NOT to use

- The tool output is unranked or already collapsed into a single gene set rather than per-case ranked gene lists
- The goal is overall ranking accuracy (e.g., rank-of-causal-gene distributions, recall at rank 20) rather than detecting recurring over-represented genes
- Analysis targets structural variants or variant-level filtering (allele frequency, pathogenicity) rather than gene-prioritization output

## Inputs

- Per-case ranked gene list output from VIBE (tab-delimited results.tsv ordered by descending highest GDA score, one file per of the 305 benchmark cases, generated without ontology traversal)
- Target gene identifiers as NCBI Gene IDs (e.g., 4204 for MECP2, 8085 for KMT2D)

## Outputs

- CSV table (e.g., mecp2_kmt2d_top10_counts.csv) with per-gene top-10 occurrence counts, reported reference values, and absolute differences

## How to apply

Load the ranked gene lists produced per case at default settings (for VIBE, the tab-delimited output ordered by descending highest GDA score per NCBI gene identifier). Parse each case's ranking (e.g., with Python/pandas), extract the top 10 NCBI gene identifiers per case, and tally how often each target gene (MECP2 = 4204, KMT2D = 8085) appears across the 305 cases. Compare observed counts against reported values (≈226 for MECP2, ≈219 for KMT2D) and write per-gene counts, reported values, and absolute differences to a CSV. Rationale: VIBE groups gene-disease associations by unique NCBI gene identifiers and ranks by highest GDA score, so a gene with extremely broad literature coverage can dominate top positions; counting top-N occurrences makes this bias measurable.

## Related tools

- **VIBE** (Source of the per-case ranked gene prioritization output (tab-delimited, sorted by highest GDA score) that is tallied for over-represented genes) — https://github.com/molgenis/vibe
- **Python** (Parsing each case's ranked gene list with pandas, extracting top-N identifiers, and tallying gene occurrences) — https://github.com/molgenis/vibe-suppl
- **R** (Benchmark result processing and plotting of the per-case gene-rank data) — https://github.com/molgenis/vibe-suppl

## Examples

```
python3 VibeBenchmarkFileGenerator.py results/ vibe_results.tsv none
```

## Evaluation signals

- Top-10 counts of MECP2 and KMT2D across the 305 cases reproduce the reported over-representation of ≈226 and ≈219 cases respectively
- Every case contributes at most one occurrence per gene to the top-10 tally, so per-gene counts cannot exceed 305
- Counts are computed on NCBI gene identifiers, not HGNC symbols, matching VIBE's output grouping
- Output CSV contains exactly three columns per gene: observed count, reported value, and absolute difference
- Spot-check a few parsed cases manually against the raw VIBE results.tsv to confirm the top-10 extraction (VIBE output is ordered by descending highest GDA score)

## Limitations

- The bias signal is tool-version dependent: the paper notes VIBE's over-representation of MECP2 and KMT2D at high positions and aims to resolve it in upcoming versions
- The benchmark used default settings without ontology traversal, so counts do not reflect behavior when -m/-n traversal options expand the phenotypic search space
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools

## Evidence

- [discussion] VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions, to be resolved in upcoming versions: "This is likely caused by a form of bias, which we naturally aim to resolve in upcoming versions"
- [methods] Genes are ranked by highest GDA score per NCBI gene identifier: [evidence span withheld — anchor status: not_found]
- [results] Benchmark scale: 305 cases with causal genes tracked to rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] Benchmark data and processing scripts are distributed via the vibe-suppl repository: [evidence span withheld — anchor status: not_found]
- [readme] VIBE default output is a ranked gene list usable for top-N tallying: "A tool to generate prioritized genes using phenotype information."
