---
name: ranked-gene-list-parsing
description: Use when you have ranked gene lists from a prioritization tool (tab-delimited,
  ordered by score) and need to quantify over-representation of specific genes (e.g.
  MECP2, KMT2D) at top ranks across many cases.
when_to_use_negative:
- The tool output is not ranked (e.g. unsorted candidate gene sets) — top-N position
  counting requires a defined ordering such as descending highest GDA score
- You need per-case rank-of-causal-gene statistics (rank distributions, recall curves)
  rather than aggregate occurrence counts of specific genes
- Ontology traversal was enabled (e.g. -n children/distance options), so counts would
  not reproduce the paper's default-settings over-representation values
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: VIBE
  role: Source of the ranked gene lists; its tab-delimited output ordered by highest
    GDA score per NCBI gene identifier is the object being parsed
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Used with pandas to parse the tab-delimited VIBE rankings and tally top-N
    gene occurrences across the 305 cases
- name: R
  role: Benchmarking results processing for the paper (e.g. PaperPlots.R in vibe-suppl)
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:3addf083a0f61ab8a569fe858043c190dddab39aaf547798b9b4ba4c59387207
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# ranked-gene-list-parsing

## Summary

Parses per-case ranked gene output files from phenotype-based prioritization tools (e.g. VIBE results ordered by descending highest GDA score per NCBI gene identifier) and tallies how often specific causal genes occur within top-N positions across a benchmark cohort of 305 rare disease patient cases.

## When to use

Use when you have ranked gene lists from a prioritization tool (tab-delimited, ordered by score) and need to quantify over-representation of specific genes (e.g. MECP2, KMT2D) at top ranks across many cases.

## When NOT to use

- The tool output is not ranked (e.g. unsorted candidate gene sets) — top-N position counting requires a defined ordering such as descending highest GDA score
- You need per-case rank-of-causal-gene statistics (rank distributions, recall curves) rather than aggregate occurrence counts of specific genes
- Ontology traversal was enabled (e.g. -n children/distance options), so counts would not reproduce the paper's default-settings over-representation values

## Inputs

- Tab-delimited ranked gene list files per case (VIBE output: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources)
- Benchmark dataset for 305 rare disease patient cases (e.g. benchmark_data-hgnc_symbol.tsv from vibe-suppl / Zenodo record 3662470)
- NCBI gene identifiers of interest (e.g. 4204 = MECP2, 8085 = KMT2D)

## Outputs

- Per-gene top-N occurrence counts across all cases (e.g. mecp2_kmt2d_top10_counts.csv with counts, reported values, and absolute differences)
- Per-case top-N NCBI gene identifier subsets extracted from each ranked list

## How to apply

Load the benchmark output gene lists (e.g. VIBE's tab-delimited output from the 305 rare disease patient cases, generated at default settings without ontology traversal, available via the molgenis/vibe-suppl benchmark repository or Zenodo record 3662470). Parse each case's ranking with Python/pandas, reading the tab-delimited VIBE output whose rows are ordered by descending highest GDA score per NCBI gene identifier. For each case, extract the top-N ranked NCBI gene identifiers (e.g. top 10, N=20 corresponds to the rank at which all tools reach their total recall of 308 causal genes in the clinical exome simulation) and tally occurrences of the genes of interest, e.g. gene 4204 (MECP2) and gene 8085 (KMT2D). Compare the observed top-N counts against the reported over-representation values (≈226 of 305 cases for MECP2 and ≈219 for KMT2D) and write per-gene counts, reported values, and absolute differences to a CSV. The counts expose a tool-level bias signal discussed by the VIBE authors as likely caused by over-representation at high rank positions.

## Related tools

- **VIBE** (Source of the ranked gene lists; its tab-delimited output ordered by highest GDA score per NCBI gene identifier is the object being parsed) — https://github.com/molgenis/vibe
- **Python** (Used with pandas to parse the tab-delimited VIBE rankings and tally top-N gene occurrences across the 305 cases)
- **R** (Benchmarking results processing for the paper (e.g. PaperPlots.R in vibe-suppl)) — https://github.com/molgenis/vibe-suppl

## Examples

```
import pandas as pd; df = pd.read_csv('results.tsv', sep='\t'); top10_ids = df.head(10)['gene (NCBI)'].tolist()
```

## Evaluation signals

- Tabulated top-10 occurrences of MECP2 and KMT2D reproduce the reported over-representation (≈226 and ≈219 of 305 cases respectively)
- Per-case parsed list length is consistent: top-N subset has exactly N gene identifiers (or fewer only for outputs with fewer genes)
- Ordering invariant: each parsed list must remain sorted by descending highest GDA score before extracting the top-N
- Count totals must not exceed the number of cases (max 305 occurrences per gene at N=1 case-level upper bound per case)
- NCBI gene ID mapping is correct: 4204 ↔ MECP2 and 8085 ↔ KMT2D, consistent with benchmark_file_conversion_data.tsv from genenames.org

## Limitations

- The over-representation of MECP2 and KMT2D reflects a tool bias the VIBE authors attribute to a form of bias they aim to resolve in upcoming versions
- Counts depend on benchmark data compatible with HPO release 2018-03-08 and the exact DisGeNET r5.0/r6.0 database version used
- Phenotype names in the benchmark data must exist within the Human Phenotype Ontology, and gene symbol-to-NCBI-ID conversion requires the HGNC conversion file; mismatches silently drop genes

## Evidence

- [methods] Output ordered by descending highest GDA score per NCBI gene identifier: [evidence span withheld — anchor status: not_found]
- [results] Rank-20 recall context for top-N choices: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [results] Figure 1 causal gene rank definition: "Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list."
- [discussion] Bias motivation for counting MECP2/KMT2D: "This is likely caused by a form of bias, which we naturally aim to resolve in upcoming versions"
- [readme] VIBE tab-delimited output schema used for parsing: "gene (NCBI)	gene symbol (HGNC)	highest GDA score	diseases (UMLS) with sources"
