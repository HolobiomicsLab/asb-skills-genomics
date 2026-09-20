---
name: taxonomic-hierarchy-sunburst-rendering
description: Use when tab-separated Kraken2 kreport2 files are available and the goal
  is interactive exploration of read distribution across taxonomic ranks (Sankey/Sunburst/species
  table) for one or more barcoded samples.
when_to_use_negative:
- Input is raw FASTQ or unclassified Kraken2 output rather than kreport2 report files
  — classification must already be done (delegated to the nanometanf pipeline).
- A static publication figure is needed rather than an interactive dashboard view;
  the Sankey/Sunburst are Dash callback-driven components, not print exports.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kraken2
  role: Produces the per-sample kreport2 taxonomic classification reports that are
    parsed and visualized
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Dash
  role: Local web-application framework hosting the Sankey, Sunburst and species-table
    tabs driven by callbacks and stores
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: pandas
  role: Aggregates per-sample report frames with groupby(['taxid','rank','name'])
    and sums '%' , 'cumul_reads', 'reads'
- name: Nanometa Live
  role: Application in which the taxonomic visualization layer (Sankey/Sunburst/tables)
    is implemented
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:47b58d13b6de977780f64f36153d1c8c62edc5aa439c963811940045048b1351
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - Dash
  - Kraken2
  - Nanometa Live
  - pandas
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# taxonomic-hierarchy-sunburst-rendering

## Summary

Render Kraken2 per-sample classification reports as interactive taxonomic visualizations — a Sankey diagram of read flow across ranks, a click-to-explore Sunburst, and a most-abundant-species table — inside the Nanometa Live Dash GUI so users can explore a sample's taxonomic composition in real time.

## When to use

Use when tab-separated Kraken2 kreport2 files are available and the goal is interactive exploration of read distribution across taxonomic ranks (Sankey/Sunburst/species table) for one or more barcoded samples.

## When NOT to use

- Input is raw FASTQ or unclassified Kraken2 output rather than kreport2 report files — classification must already be done (delegated to the nanometanf pipeline).
- A static publication figure is needed rather than an interactive dashboard view; the Sankey/Sunburst are Dash callback-driven components, not print exports.

## Inputs

- Per-sample Kraken2 kreport2 files (tab-separated; columns '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name') in results/kraken2
- Dash stores 'app-config' and 'selected-sample' for barcode/sample selection

## Outputs

- Interactive Sankey diagram of sequencing-read flow across taxonomic ranks
- Interactive Sunburst radial plot with click-to-explore navigation
- Most-abundant-species table restricted to species/subspecies ranks ('S','S1','S2','S3')

## How to apply

Load per-sample Kraken2 reports from results/kraken2 with the load_kraken_data loader, parsing the kreport2 columns '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name'. Aggregate per-sample frames in pandas via groupby(['taxid','rank','name'], as_index=False) summing '%', 'cumul_reads' and 'reads' for the 'All Samples' view, while keeping per-sample slices for the barcode selector. For the most-abundant-species table restrict to SPECIES_RANKS = frozenset({'S','S1','S2','S3'}) so subspecies are counted but never double-summed with their own children. Build a clade hierarchy from cumul_reads (node plus descendants) and render both a Sankey diagram of read flow across ranks and a Sunburst with click-to-explore, exposing them through Dash callbacks consuming the 'app-config' and 'selected-sample' stores, with filter options in a collapsed-by-default section.

## Related tools

- **Kraken2** (Produces the per-sample kreport2 taxonomic classification reports that are parsed and visualized) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Dash** (Local web-application framework hosting the Sankey, Sunburst and species-table tabs driven by callbacks and stores) — https://github.com/FOI-Bioinformatics/nanometa_live
- **pandas** (Aggregates per-sample report frames with groupby(['taxid','rank','name']) and sums '%' , 'cumul_reads', 'reads')
- **Nanometa Live** (Application in which the taxonomic visualization layer (Sankey/Sunburst/tables) is implemented) — https://github.com/FOI-Bioinformatics/nanometa_live

## Examples

```
df = pd.read_csv('results/kraken2/sample.kreport2', sep='\t', names=['%','cumul_reads','reads','rank','taxid','name']); agg = df.groupby(['taxid','rank','name'], as_index=False)[['%','cumul_reads','reads']].sum()
```

## Evaluation signals

- Module runs end-to-end on the sample kreport2 data without errors.
- Rendered Sankey and Sunburst figures are consistent with the aggregate pandas frame.
- Species-table invariant holds: species.reads + sum(children.reads) == species.cumul_reads.
- Subspecies rows ('S1','S2','S3') are counted but never summed twice with their parent species.
- Barcode selector correctly switches between per-sample slices and the aggregated 'All Samples' view via the 'selected-sample' store.

## Limitations

- Hierarchy reconstruction relies on cumul_reads being a correct clade total (node plus descendants); corrupt or non-cumulative kreport2 files will break the Sankey/Sunburst consistency.
- Filter options are in a section collapsed by default, so users may miss rank filtering controls.
- Subspecies ranks must be handled explicitly via SPECIES_RANKS; other nonstandard rank codes could be mis-aggregated.

## Evidence

- [results] Sankey tab interactive: [evidence span withheld — anchor status: not_found]
- [full_text] report columns: [evidence span withheld — anchor status: not_found]
- [full_text] species ranks: [evidence span withheld — anchor status: not_found]
- [full_text] validation invariant: [evidence span withheld — anchor status: not_found]
- [readme] readme interactive visualisations: [evidence span withheld — anchor status: external_doc]
