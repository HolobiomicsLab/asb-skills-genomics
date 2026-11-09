---
name: gene-rank-frequency-quantification
description: Use when benchmarking phenotype-based gene prioritization tools and you
  need to detect systematic bias, e.g. certain genes appearing disproportionately
  often in top-10/top-20 ranked output positions across many patient cases.
when_to_use_negative:
- When the goal is overall ranking accuracy (e.g. causal gene rank distributions or
  recall curves) rather than detecting over-representation of specific recurring genes
- When tool outputs are not per-case ranked gene lists ordered by a single score (e.g.
  unranked candidate sets or rank-stabilized median-of-25-permutation outputs, which
  need a different handling)
- When diagnosing why a single case ranked poorly rather than quantifying cohort-level
  gene-frequency bias
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0089
tools:
- name: VIBE
  role: Source of the per-case ranked gene lists (tab-delimited, ordered by descending
    highest GDA score per NCBI gene identifier) whose top-N positions are tallied;
    the skill quantifies its MECP2/KMT2D over-representation bias
  repo: https://github.com/molgenis/vibe
- name: Python
  role: Pandas-based parsing of each case's gene ranking and tallying of top-10 occurrences
    of MECP2 (4204) and KMT2D (8085)
- name: R
  role: Downstream processing/plotting of benchmarking results (PaperPlots.R in vibe-suppl)
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d2d9632b88805bd56bd4946335aafe02195cbaf2b12bab8b7333b29120f0aa0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gene-rank-frequency-quantification

## Summary

Quantifies over-representation of specific genes (e.g. MECP2, NCBI 4204; KMT2D, NCBI 8085) by counting how often each causal gene appears within the top N ranked positions of a gene prioritization tool's output across a benchmark cohort of 305 rare disease patient cases, exposing rank bias in tools like VIBE.

## When to use

Use when benchmarking phenotype-based gene prioritization tools and you need to detect systematic bias, e.g. certain genes appearing disproportionately often in top-10/top-20 ranked output positions across many patient cases.

## When NOT to use

- When the goal is overall ranking accuracy (e.g. causal gene rank distributions or recall curves) rather than detecting over-representation of specific recurring genes
- When tool outputs are not per-case ranked gene lists ordered by a single score (e.g. unranked candidate sets or rank-stabilized median-of-25-permutation outputs, which need a different handling)
- When diagnosing why a single case ranked poorly rather than quantifying cohort-level gene-frequency bias

## Inputs

- Per-case ranked gene list output from a gene prioritization tool (e.g. VIBE results.tsv ordered by descending highest GDA score per NCBI gene identifier)
- Benchmark case set of 305 rare disease patient cases with HPO codes (benchmark_data.tsv with ID and comma-separated phenotypes)
- NCBI gene identifiers of genes to tally (e.g. 4204 for MECP2, 8085 for KMT2D)

## Outputs

- CSV of per-gene top-N occurrence counts, reported values, and absolute differences (e.g. mecp2_kmt2d_top10_counts.csv)
- Tallied occurrence counts of specific genes within top-10/top-20 positions across all benchmark cases

## How to apply

Load each tool's per-case ranked gene list output (for VIBE, the tab-delimited results ordered by descending highest GDA score per NCBI gene identifier, generated at default settings without ontology traversal). Parse each case's ranking with Python/pandas, extract the top 10 (or top 20) ranked NCBI gene identifiers, and tally occurrences of the gene(s) of interest across all 305 cases. Compare the observed top-N counts against reported values (MECP2 ≈226 and KMT2D ≈219 of 305 cases for VIBE) and record absolute differences per gene; optionally extend to counting the rank of the causal gene itself, since at rank 20 all tools reach their total recall of 308 causal genes in the clinical exome simulation. Write per-gene counts, reported values, and differences to a CSV (e.g. mecp2_kmt2d_top10_counts.csv). This top-N occurrence counting is what surfaced VIBE's high-position bias, which the authors attribute to a form of bias to be resolved in upcoming versions.

## Related tools

- **VIBE** (Source of the per-case ranked gene lists (tab-delimited, ordered by descending highest GDA score per NCBI gene identifier) whose top-N positions are tallied; the skill quantifies its MECP2/KMT2D over-representation bias) — https://github.com/molgenis/vibe
- **Python** (Pandas-based parsing of each case's gene ranking and tallying of top-10 occurrences of MECP2 (4204) and KMT2D (8085))
- **R** (Downstream processing/plotting of benchmarking results (PaperPlots.R in vibe-suppl)) — https://github.com/molgenis/vibe-suppl

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Tallied top-10 occurrences reproduce the reported over-representation: MECP2 ≈226 and KMT2D ≈219 of 305 cases (accept small absolute differences per gene)
- Counts are bounded by the number of cases (each gene can appear at most once per case's top-10, so 0 ≤ count ≤ 305)
- Parsed rankings are consistent with VIBE's output schema: NCBI gene identifier, HGNC symbol, highest GDA score, diseases with sources, sorted by descending highest GDA score
- Cross-check with the benchmark's rank-20 invariant: all tools reach their total recall of 308 causal genes at rank 20, so top-20 tallies of causal genes should not exceed 308

## Limitations

- The bias signal (MECP2, KMT2D over-representation) is tool-specific to VIBE's DisGeNET-backed scoring and is expected to change in upcoming VIBE versions, so counts must be re-derived per tool version and database release
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- VIBE results must be generated at default settings without ontology traversal (-n/-m/-w options unused); enabling child or distance-based ontology traversal changes the search space and invalidates comparison to reported counts

## Evidence

- [results] Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list. The number of missed genes for each tool, where the causal gene was not present in the: "Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list. The number of missed genes for each tool, where the causal gene was not present in the"
- [results] At rank 20, all tools arrive at their total recall of 308 causal genes.: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [methods] The GDAs are grouped by unique NCBI gene identifiers: [evidence span withheld — anchor status: not_found]
- [discussion] VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions: "This is likely caused by a form of bias, which we naturally aim to resolve in upcoming versions"
- [readme] −n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space: "Using the user-defined phenotypes and phenotypes that are related to them with a maximum distance of 1"
