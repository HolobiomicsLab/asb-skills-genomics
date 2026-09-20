---
name: taxonomic-read-flow-sankey-visualization
description: Use when per-sample Kraken2 kreport2 files are available and the goal
  is to explore a sample's taxonomic composition interactively, e.g. inspecting how
  sequencing reads distribute across rank levels during an Oxford Nanopore run.
when_to_use_negative:
- Input is not Kraken2 report output (e.g. raw FASTQ, minimap2 coverage files, or
  BLAST results) — the loader expects kreport2 columns
- The goal is per-sample quality control (read length, filtering statistics) rather
  than taxonomic composition — use the QC panel instead
- No GUI/dashboard interaction is needed and only numeric abundance tables are required
  — a plain pandas aggregation suffices
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: Kraken2
  role: Produces the per-sample kreport2 classification reports that feed the Sankey/Sunburst
    hierarchy
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Dash
  role: Framework for the single-page local web app whose callbacks render the Sankey,
    Sunburst and species table
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Nanometa Live
  role: The GUI application whose Sankey tab displays the read-flow visualizations
    built from nanometanf results
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: pandas
  role: Aggregates per-sample kreport2 frames via groupby(['taxid','rank','name'])
    and computes clade counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:b378c02c1b7b712b71872d03609ab57e90f03a8975291baff2a4b1e0ced1a720
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - Dash
  - Kraken2
  - Nanometa Live
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# taxonomic-read-flow-sankey-visualization

## Summary

Render Kraken2 per-sample taxonomic classification output from the Nanometa Live/nanometanf pipeline as interactive read-flow visualizations — a Sankey diagram of read distribution across taxonomic ranks, a click-to-explore Sunburst from the same clade hierarchy, and a most-abundant-species table — inside the Dash GUI.

## When to use

Use when per-sample Kraken2 kreport2 files are available and the goal is to explore a sample's taxonomic composition interactively, e.g. inspecting how sequencing reads distribute across rank levels during an Oxford Nanopore run.

## When NOT to use

- Input is not Kraken2 report output (e.g. raw FASTQ, minimap2 coverage files, or BLAST results) — the loader expects kreport2 columns
- The goal is per-sample quality control (read length, filtering statistics) rather than taxonomic composition — use the QC panel instead
- No GUI/dashboard interaction is needed and only numeric abundance tables are required — a plain pandas aggregation suffices

## Inputs

- Per-sample Kraken2 kreport2 report files (tab-separated: %, cumul_reads, reads, rank, taxid, name) from results/kraken2
- Barcode/sample selector state and app configuration held in Dash 'app-config' and 'selected-sample' stores

## Outputs

- Interactive Sankey diagram of read flow across taxonomic ranks
- Radial Sunburst plot of the same clade hierarchy with click-to-explore interaction
- Most-abundant-species table restricted to species-level ranks (S, S1, S2, S3)
- 'All Samples' aggregate frame plus per-sample slices per barcode

## How to apply

Load per-sample Kraken2 reports from the results/kraken2 directory via the load_kraken_data loader, parsing the tab-separated kreport2 columns '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name'. Aggregate per-sample frames in pandas with groupby(['taxid','rank','name'], as_index=False), summing '%', 'cumul_reads' and 'reads' for an 'All Samples' view while retaining per-sample slices for the barcode selector. Build the clade hierarchy from cumul_reads (node plus descendants) so that a species' clade count equals its own reads plus all children; for the abundant-species table restrict rows with SPECIES_RANKS = frozenset({'S','S1','S2','S3'}) so subspecies are counted but never double-summed with their own children. Render the Sankey diagram of read flow across ranks and a Sunburst from the same hierarchy with click-to-explore interaction, expose all three via Dash callbacks consuming the 'app-config' and 'selected-sample' stores, and keep filter options in a collapsible section collapsed by default.

## Related tools

- **Kraken2** (Produces the per-sample kreport2 classification reports that feed the Sankey/Sunburst hierarchy) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Dash** (Framework for the single-page local web app whose callbacks render the Sankey, Sunburst and species table) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Nanometa Live** (The GUI application whose Sankey tab displays the read-flow visualizations built from nanometanf results) — https://github.com/FOI-Bioinformatics/nanometa_live
- **pandas** (Aggregates per-sample kreport2 frames via groupby(['taxid','rank','name']) and computes clade counts)

## Examples

```
agg = pd.concat(per_sample_frames).groupby(['taxid','rank','name'], as_index=False)[['%','cumul_reads','reads']].sum()
```

## Evaluation signals

- Invariant check: species.reads + sum(children.reads) == species.cumul_reads for every species row in the rendered figures and table
- Module runs end-to-end on sample kreport2 data without errors, producing Sankey, Sunburst and species-table outputs
- Rendered Sankey and Sunburst figures are numerically consistent with the aggregate pandas frame (grouped sums of '%', 'cumul_reads', 'reads')
- Species table contains only rows whose rank is in SPECIES_RANKS = frozenset({'S','S1','S2','S3'})
- Dash callbacks correctly resolve both the 'All Samples' aggregate view and per-barcode slices from the 'selected-sample' store

## Limitations

- Hierarchy counts derive from cumul_reads (node plus descendants), so subspecies (S1, S2, S3) are counted but must not be summed with their own children or clade counts are double-counted
- Depends on the exact kreport2 column layout ('%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name'); other classifier report formats will not load
- Visualization is part of a local single-page Dash app and depends on per-sample Kraken2 reports produced by the nanometanf pipeline layout (results/kraken2)

## Evidence

- [methods] Sankey tab represents read distribution: [evidence span withheld — anchor status: not_found]
- [methods] Clade hierarchy and Sankey rendering: [evidence span withheld — anchor status: not_found]
- [methods] Species rank restriction: [evidence span withheld — anchor status: not_found]
- [readme] Interactive taxonomic visualisations in README: [evidence span withheld — anchor status: external_doc]
