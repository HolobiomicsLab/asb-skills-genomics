---
name: go-term-redundancy-filtering
description: Use when a tappAS functional enrichment (partially-overlapping-samples
  z-test, FDR < 0.05) yields a large GO term list — e.g. 59 terms enriched in coDIU
  vs DIU genes — that must be condensed to non-redundant terms before reporting.
when_to_use_negative:
- The GO term list has not been statistically filtered by the tappAS enrichment (FDR
  < 0.05) and is a raw annotation vocabulary, not an enriched set.
- The goal is positional functional feature enrichment rather than GO terms — the
  91 positional functional features are reported directly without Revigo.
- The input is already a deduplicated or manually curated GO term list where Revigo
  filtering would discard meaningful term distinctions.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: Revigo
  role: Web service used to remove redundant GO terms from the enriched coDIU GO term
    list, yielding 26 unique terms
- name: tappAS
  role: Runs the functional enrichment analysis (partially-overlapping-samples z-test)
    that produces the GO term list filtered before Revigo submission
  repo: https://github.com/ConesaLab/tappAS
- name: acorde
  role: Defines the coDIU vs DIU gene sets and provides the functionally-annotated
    transcriptome (IsoAnnotLite, tappAS-compatible) that feeds the enrichment
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Environment in which the acorde package and enrichment workflow are executed
    and results are written to the TSV output
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:7a4041ea1fa37d0885ef5edcda023c8be4d09186e513c5839bce92b4db7440a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# go-term-redundancy-filtering

## Summary

Reduces a list of significantly enriched Gene Ontology terms to a non-redundant set by submitting it to Revigo, so that coDIU functional enrichment results (59 enriched GO terms) are reported as interpretable, non-overlapping terms (26 unique GO terms).

## When to use

Use when a tappAS functional enrichment (partially-overlapping-samples z-test, FDR < 0.05) yields a large GO term list — e.g. 59 terms enriched in coDIU vs DIU genes — that must be condensed to non-redundant terms before reporting.

## When NOT to use

- The GO term list has not been statistically filtered by the tappAS enrichment (FDR < 0.05) and is a raw annotation vocabulary, not an enriched set.
- The goal is positional functional feature enrichment rather than GO terms — the 91 positional functional features are reported directly without Revigo.
- The input is already a deduplicated or manually curated GO term list where Revigo filtering would discard meaningful term distinctions.

## Inputs

- Filtered list of GO terms significantly enriched in coDIU vs DIU genes at FDR < 0.05 (59 terms)
- tappAS functional enrichment output with Benjamini-Hochberg FDR values
- Gene-to-condition table assigning genes to coDIU and DIU sets

## Outputs

- Non-redundant set of 26 unique GO terms after Revigo filtering
- codiu_functional_enrichment.tsv containing positional feature and GO term enrichment results with FDR values

## How to apply

First run the tappAS functional enrichment comparing coDIU genes against DIU genes with the partially overlapping samples z-test, correcting p-values with Benjamini-Hochberg and retaining features at FDR < 0.05 (this recovers 59 GO terms in the acorde mouse study). Submit only this filtered, statistically enriched GO term list to Revigo, not the raw annotation vocabulary. Verify that Revigo removes redundancy leaving exactly 26 unique GO terms for the acorde coDIU dataset. Finally, write both the positional feature results and the deduplicated GO term results with their FDR values to codiu_functional_enrichment.tsv.

## Related tools

- **Revigo** (Web service used to remove redundant GO terms from the enriched coDIU GO term list, yielding 26 unique terms)
- **tappAS** (Runs the functional enrichment analysis (partially-overlapping-samples z-test) that produces the GO term list filtered before Revigo submission) — https://github.com/ConesaLab/tappAS
- **acorde** (Defines the coDIU vs DIU gene sets and provides the functionally-annotated transcriptome (IsoAnnotLite, tappAS-compatible) that feeds the enrichment) — https://github.com/ConesaLab/acorde
- **R** (Environment in which the acorde package and enrichment workflow are executed and results are written to the TSV output) — https://github.com/ConesaLab/acorde

## Evaluation signals

- Revigo output contains exactly 26 unique GO terms when run on the acorde coDIU enriched list (input = 59 GO terms at FDR < 0.05).
- Every GO term submitted to Revigo passes the Benjamini-Hochberg FDR < 0.05 threshold in the tappAS coDIU vs DIU enrichment.
- Output TSV (codiu_functional_enrichment.tsv) includes both the 91 positional functional features and the deduplicated GO term results, each with FDR values.
- The deduplicated term count is strictly smaller than the enriched input list, confirming redundancy was actually removed.
- Positional functional features are not passed through Revigo — they should be unchanged in the output.

## Limitations

- Revigo deduplication depends on which GO term in a redundant cluster is retained, so the 26-term set may vary slightly with Revigo settings or GO database version.
- Functional analyses rely on pre-annotated references transferred via IsoAnnotLite (v2.6) from tappAS' mouse RefSeq 78 annotation; a de novo annotation tool was not available at the time.
- The acorde R package does not include the long-read transcriptome building or the functional analysis steps themselves — these are performed with external tools (tappAS, Revigo).

## Evidence

- [methods] Revigo deduplicates the enriched GO list to 26 unique terms: [evidence span withheld — anchor status: not_found]
- [results] Enrichment result the deduplication is applied to: [evidence span withheld — anchor status: not_found]
- [readme] tappAS provides the functional analysis framework upstream of GO filtering: "To couple these analysis with a biologically interpretable readout, we incorporate functional annotations onto isoform models, and use [tappAS](https://github.com/ConesaLab/tappAS) for functional"
- [readme] IsoAnnotLite transfer of tappAS annotations onto long-read transcriptomes: [evidence span withheld — anchor status: external_doc]
