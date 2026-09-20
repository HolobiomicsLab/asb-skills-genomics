---
name: dash-callback-dashboard-integration
description: Use when building or extending a Dash-based visualization layer that
  must react to per-sample selections and app configuration, e.g. rendering Kraken2
  kreport2 read distributions as Sankey/Sunburst figures.
when_to_use_negative:
- The task is taxonomy classification itself (Kraken2/BLAST analysis) — this skill
  only renders already-produced kreport2 output, which is delegated to the nanometanf
  pipeline.
- Input is raw sequencing data (FASTQ) or per-sample QC statistics rather than taxonomic
  classification reports.
- A non-Dash visualization stack is mandated (e.g. static plotting to file with no
  interactivity or per-sample store-driven updates).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: Nanometa Live
  role: Front-end Dash web application whose taxonomic visualization layer (Sankey,
    Sunburst, species tables) this skill implements
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Dash
  role: Framework used to implement the local single-page web app with callbacks consuming
    'app-config' and 'selected-sample' stores
- name: Kraken2
  role: Produces the per-sample kreport2 classification reports that are parsed and
    visualized
- name: pandas
  role: Used to aggregate per-sample report frames via groupby(['taxid','rank','name'])
    and sum '%'/'cumul_reads'/'reads'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:07bdfda8689805539496b972ff6988fa646a74c5cbcb8c1fb6e050e4f01ffd56
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dash-callback-dashboard-integration

## Summary

Wire taxonomic visualizations (Sankey, Sunburst, abundance tables) into a local Dash web application so that classification output from per-sample Kraken2 reports is rendered and updated interactively in Nanometa Live.

## When to use

Use when building or extending a Dash-based visualization layer that must react to per-sample selections and app configuration, e.g. rendering Kraken2 kreport2 read distributions as Sankey/Sunburst figures.

## When NOT to use

- The task is taxonomy classification itself (Kraken2/BLAST analysis) — this skill only renders already-produced kreport2 output, which is delegated to the nanometanf pipeline.
- Input is raw sequencing data (FASTQ) or per-sample QC statistics rather than taxonomic classification reports.
- A non-Dash visualization stack is mandated (e.g. static plotting to file with no interactivity or per-sample store-driven updates).

## Inputs

- Per-sample Kraken2 report files (tab-separated kreport2: '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name') from the results/kraken2 directory
- Dash stores: 'app-config' and 'selected-sample'
- Barcode/sample selector values for multiplexed runs

## Outputs

- Interactive Sankey diagram of read flow across taxonomic ranks
- Interactive Sunburst radial plot of the clade hierarchy with click-to-explore
- Most-abundant-species table restricted to ranks S, S1, S2, S3

## How to apply

Load per-sample Kraken2 reports from results/kraken2 via a load_kraken_data loader, parsing the tab-separated kreport2 columns '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name'. Aggregate per-sample frames in pandas with groupby(['taxid','rank','name'], as_index=False), summing '%', 'cumul_reads' and 'reads' for the 'All Samples' view while keeping per-sample slices for the barcode selector. Build a clade hierarchy from cumul_reads (node plus descendants), then render a Sankey diagram of read flow across taxonomic ranks and a Sunburst radial plot from the same hierarchy with click-to-explore interaction. Restrict the most-abundant-species table to SPECIES_RANKS = frozenset({'S','S1','S2','S3'}) so subspecies are counted but never double-summed with their own children. Expose the three visualizations through Dash callbacks consuming the 'app-config' and 'selected-sample' stores, with filter options in a collapsible section collapsed by default.

## Related tools

- **Nanometa Live** (Front-end Dash web application whose taxonomic visualization layer (Sankey, Sunburst, species tables) this skill implements) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Dash** (Framework used to implement the local single-page web app with callbacks consuming 'app-config' and 'selected-sample' stores)
- **Kraken2** (Produces the per-sample kreport2 classification reports that are parsed and visualized)
- **pandas** (Used to aggregate per-sample report frames via groupby(['taxid','rank','name']) and sum '%'/'cumul_reads'/'reads')

## Examples

```
df.groupby(['taxid','rank','name'], as_index=False)[['%','cumul_reads','reads']].sum()  # aggregate kreport2 frames for 'All Samples' Sankey/Sunburst
```

## Evaluation signals

- The module runs end-to-end on the sample kreport2 data without errors.
- Rendered Sankey and Sunburst figures are consistent with the aggregate frame: species.reads + sum(children.reads) == species.cumul_reads.
- Species table rows are restricted to rank codes in frozenset({'S','S1','S2','S3'}) and subspecies are counted but never summed alongside their own children.
- Dash callbacks correctly react to changes in the 'app-config' and 'selected-sample' stores, switching between 'All Samples' aggregation and per-sample slices.
- Filter options render as a collapsible section that is collapsed by default.

## Limitations

- Rendering depends on the layout of the nanometanf canonical output (per-sample Kraken2 reports under results/kraken2); changes to kreport2 column layout or output paths break the loader.
- Multiplexed (barcoded) runs require per-sample slices in addition to the 'All Samples' aggregate; forgetting to retain them silently removes the barcode selector view.
- Double-counting risk if subspecies ranks (S1, S2, S3) are summed together with their parent species instead of relying on cumul_reads for clade totals.

## Evidence

- [methods] Sankey/Sunburst construction from kreport2 hierarchy: [evidence span withheld — anchor status: not_found]
- [methods] Species rank restriction: [evidence span withheld — anchor status: not_found]
- [readme] Interactive visualizations feature: [evidence span withheld — anchor status: external_doc]
- [abstract] Application implementation framework: "implemented as a local web application using the Dash framework"
- [methods] Validation invariant: [evidence span withheld — anchor status: not_found]
