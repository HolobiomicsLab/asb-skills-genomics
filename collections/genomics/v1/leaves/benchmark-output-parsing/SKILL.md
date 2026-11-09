---
name: benchmark-output-parsing
description: Use when raw tab-delimited gene-prioritizer outputs (VIBE's gene (NCBI)
  / gene symbol (HGNC) / highest GDA score columns) must be merged and normalized
  into per-case ranked benchmark files with consistent NCBI gene identifiers.
when_to_use_negative:
- The input files are not tab-delimited prioritizer outputs with the gene (NCBI)/gene
  symbol (HGNC)/highest GDA score schema (e.g., raw VCFs or variant-level annotations)
- 'Benchmarking PhenoTips: its stand-alone version is obsolete (paid cloud subscription
  only since January 2020), so its output-parsing path cannot be reproduced'
edam_operation: http://edamontology.org/operation_0226
tools:
- name: VIBE
  role: Produces the per-case tab-delimited ranked gene outputs (highest GDA score,
    UMLS disease associations) that this skill parses
  repo: https://github.com/molgenis/vibe
- name: vibe-suppl benchmark scripts
  role: VibeBenchmarkFileGenerator.py and BenchmarkFileGeneSymbolToIdConverter.py
    merge per-case outputs and convert HGNC symbols to NCBI gene IDs
  repo: https://github.com/molgenis/vibe-suppl
- name: Java 8
  role: Runtime used to invoke the VIBE jar that generates the parsed output files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:75f7b4d8d2fea98dc1cca272c81f162dfaa3a2d7f2ef58110117aa2b973b2df4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - HPO v2018-03-08
  - Java 8
  - VIBE
  - vibe-suppl benchmark scripts
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-output-parsing

## Summary

Parses VIBE's tab-delimited per-case gene rankings (NCBI gene ID, HGNC symbol, highest GDA score, UMLS disease associations) into normalized benchmark result files, converting HGNC symbols to NCBI gene IDs so causal-gene ranks can be scored across 305 patient cases.

## When to use

Use when raw tab-delimited gene-prioritizer outputs (VIBE's gene (NCBI) / gene symbol (HGNC) / highest GDA score columns) must be merged and normalized into per-case ranked benchmark files with consistent NCBI gene identifiers.

## When NOT to use

- The input files are not tab-delimited prioritizer outputs with the gene (NCBI)/gene symbol (HGNC)/highest GDA score schema (e.g., raw VCFs or variant-level annotations)
- Benchmarking PhenoTips: its stand-alone version is obsolete (paid cloud subscription only since January 2020), so its output-parsing path cannot be reproduced

## Inputs

- VIBE per-case tab-delimited output files (columns: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources)
- benchmark_data.tsv (case ID plus comma-separated HPO phenotype names compatible with HPO release 2018-03-08)
- benchmark_file_conversion_data.tsv (HGNC symbols/IDs to NCBI gene IDs from genenames.org)

## Outputs

- Merged vibe_results.tsv with one ranked gene list per patient case
- Per-case causal-gene ranks against the known causal genes (308 causal genes across 305 cases)
- Tool-wide rank statistics for benchmark comparison (Figure 1/2-style outputs via PaperPlots.R)

## How to apply

After running VIBE per patient case (default settings, no -w/-m/-n ontology traversal), collect the per-case tab-delimited outputs where genes are ranked in descending order of highest GDA score and disease associations are pipe-separated UMLS terms with GDA scores and PubMed evidence IDs. Run VibeBenchmarkFileGenerator.py over the results directory to produce a merged vibe_results.tsv. Then convert HGNC symbols to NCBI gene IDs using BenchmarkFileGeneSymbolToIdConverter.py with the genenames.org-generated benchmark_file_conversion_data.tsv (containing HGNC IDs, approved/previous symbols, and NCBI IDs), which is required because other tools in the benchmark (AMELIE, hiPHIVE, PhenIX, GADO, Phenomizer, PubCaseFinder) report HGNC symbols while the benchmark evaluates against NCBI gene IDs. Finally, locate each of the 308 causal genes within its case's ranking and compute per-case rank statistics; the paper expects total recall of all 308 causal genes by rank 20.

## Related tools

- **VIBE** (Produces the per-case tab-delimited ranked gene outputs (highest GDA score, UMLS disease associations) that this skill parses) — https://github.com/molgenis/vibe
- **vibe-suppl benchmark scripts** (VibeBenchmarkFileGenerator.py and BenchmarkFileGeneSymbolToIdConverter.py merge per-case outputs and convert HGNC symbols to NCBI gene IDs) — https://github.com/molgenis/vibe-suppl
- **Java 8** (Runtime used to invoke the VIBE jar that generates the parsed output files)

## Examples

```
python3 VibeBenchmarkFileGenerator.py results/ vibe_results.tsv none
```

## Evaluation signals

- Each per-case output is a tab-delimited gene list whose rows are sorted in descending order of highest GDA score
- All HGNC symbols in the merged results resolve to NCBI gene IDs via benchmark_file_conversion_data.tsv (approved plus previously used symbols)
- Every one of the 308 causal genes can be located within its case's ranking (expected total recall by rank 20)
- Merged vibe_results.tsv contains one ranked list for each of the 305 patient cases with no missing results directories
- Downstream PaperPlots.R consumes the generated files exactly as produced from the Zenodo-recorded commits (zenodo.org/record/3662470)

## Limitations

- The vibe-suppl scripts were written assuming exact intended usage; misapplied inputs can yield 'weird behavior' despite built-in checks
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution and shows a bias with over-represented genes (e.g., MECP2, KMT2D) at high positions, affecting rank-based benchmark interpretation
- Benchmark results are representative of clinical practice only to a degree and do not demonstrate individual strengths of the tested tools

## Evidence

- [readme] output columns: "gene (NCBI)	gene symbol (HGNC)	highest GDA score	diseases (UMLS) with sources"
- [readme] merged generator: [evidence span withheld — anchor status: not_found]
- [readme] symbol conversion: [evidence span withheld — anchor status: not_found]
- [methods] benchmark setup: "The resulting HPO codes were supplied to VIBE and seven other available tools that can prioritize genes based on phenotypes"
- [results] recall expectation: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] script fragility: [evidence span withheld — anchor status: not_found]
