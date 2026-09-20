---
name: long-read-isoform-database-querying
description: Use when you have a populated isoSeQL SQLite database (isoform, counts,
  isoform_ends, ends_counts, sampleData, exp tables) and need to compare isoforms,
  counts, or splice junction ends across multiple Iso-Seq samples.
when_to_use_negative:
- Input is raw subreads or unannotated HiFi reads — the upstream phase (i) database
  population (isoseq > minimap2/pbmm2 > cDNA_Cupcake > SQANTI3 + sqanti3_RulesFilter.py)
  must run first
- Only a single sample exists with no cross-sample comparison needed — the mismatched
  PB.X.Y identifier problem and intersection plots only matter across multiple datasets
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3512
tools:
- name: isoSeQL
  role: Provides the pre-written phase-(ii) query functions against the consolidated
    SQLite database; users can also write custom queries directly
  repo: https://github.com/christine-liu/isoSeQL
- name: Python 3.9.6
  role: Runtime for isoSeQL query functions and for consolidating isoform records
    and writing count matrices with pandas and numpy
- name: plotly
  role: Renders reads/isoforms-per-structural-category bar plots exported as PNG
- name: ComplexUpset
  role: Builds cross-sample intersection UpSet plots from per-sample isoform lists
    exported from SQLite
  repo: https://github.com/krassowski/complex-upset
- name: R 4.1.1
  role: Environment (v 4.1.1) used with ComplexUpset to generate UpSet plots
- name: SQANTI3
  role: Source of the filtered classification records loaded into the database; structural
    categories drive the count aggregation
  repo: https://github.com/ConesaLab/SQANTI3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:030cc15c0995fdbdaa3e3b98c77a6fb9bf92c01f02be6fe2e4d2070e0fc82004
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - ComplexUpset
  - Python
  - Python 3.9.6
  - R
  - R 4.1.1
  - SQANTI3
  - isoSeQL
  - numpy
  - optparse
  - pandas
  - plotly
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# long-read-isoform-database-querying

## Summary

Phase (ii) of isoSeQL usage: query a consolidated SQLite database built from SQANTI3-filtered classification files to extract sample metadata, isoform-by-sample count matrices, structural-category plots, cross-sample UpSet plots, and start/end read distributions, or write custom SQL queries against the database.

## When to use

Use when you have a populated isoSeQL SQLite database (isoform, counts, isoform_ends, ends_counts, sampleData, exp tables) and need to compare isoforms, counts, or splice junction ends across multiple Iso-Seq samples.

## When NOT to use

- Input is raw subreads or unannotated HiFi reads — the upstream phase (i) database population (isoseq > minimap2/pbmm2 > cDNA_Cupcake > SQANTI3 + sqanti3_RulesFilter.py) must run first
- Only a single sample exists with no cross-sample comparison needed — the mismatched PB.X.Y identifier problem and intersection plots only matter across multiple datasets

## Inputs

- Populated isoSeQL SQLite database (tables: isoform, counts, isoform_ends, ends_counts, sampleData, exp) built from SQANTI3-filtered classification files
- User-specified intron chain for start/end coordinate queries
- Per-sample SQANTI3-filtered classification records loaded into the database

## Outputs

- Isoform-by-sample count matrix CSV
- Sample metadata extract
- Bar plots of reads/isoforms per SQANTI3 structural category (PNG, rendered with plotly)
- Cross-sample intersection UpSet plots (PNG, built in R with ComplexUpset)
- Start/end read distribution plots for a specified intron chain (PNG)
- Results of custom SQLite queries

## How to apply

Open the SQLite database built from SQANTI3-filtered classification files and inspect the isoform, counts, isoform_ends, ends_counts, sampleData, and exp tables. Query and consolidate isoform records with Python 3.9.6 (pandas, numpy), resolving mismatched PB.X.Y identifiers across experiments, and write the isoform-by-sample count matrix to CSV. Aggregate counts by SQANTI3 structural category and render reads/isoforms-per-category bar plots with plotly, exporting each as PNG. Export per-sample isoform lists from SQLite and build cross-sample intersection UpSet plots in R 4.1.1 with ComplexUpset; for a user-specified intron chain, retrieve start/end coordinate records from isoform_ends and ends_counts and plot read distributions across the variable ends.

## Related tools

- **isoSeQL** (Provides the pre-written phase-(ii) query functions against the consolidated SQLite database; users can also write custom queries directly) — https://github.com/christine-liu/isoSeQL
- **Python 3.9.6** (Runtime for isoSeQL query functions and for consolidating isoform records and writing count matrices with pandas and numpy)
- **plotly** (Renders reads/isoforms-per-structural-category bar plots exported as PNG)
- **ComplexUpset** (Builds cross-sample intersection UpSet plots from per-sample isoform lists exported from SQLite) — https://github.com/krassowski/complex-upset
- **R 4.1.1** (Environment (v 4.1.1) used with ComplexUpset to generate UpSet plots)
- **SQANTI3** (Source of the filtered classification records loaded into the database; structural categories drive the count aggregation) — https://github.com/ConesaLab/SQANTI3

## Examples

```
import pandas as pd, sqlite3; con = sqlite3.connect('isoSeQL.db'); counts = pd.read_sql_query('SELECT * FROM counts', con)
```

## Evaluation signals

- Every query function executes end-to-end on the example database without errors
- The isoform-by-sample count matrix CSV is produced and populated with records from all loaded experiments (PB.X.Y identifiers correctly resolved across experiments)
- Structural-category bar plot PNGs, UpSet plot PNGs, and read-distribution PNGs are generated and populated with records from all loaded experiments
- UpSet intersections in R match the per-sample isoform lists exported from SQLite
- Start/end read distribution plot for the user-specified intron chain contains coordinate records consistent with isoform_ends and ends_counts tables

## Limitations

- Phase-(ii) querying depends on the database having been built with the SQANTICL-modified SQANTI3 filtering step; the mismatched PB.X.Y identifiers must be resolved during consolidation
- The 'mega sample' concatenation alternative (isoseq > minimap2 > cDNA_cupcake > SQANTI2 on concatenated reads) does not scale well with increased sample numbers or SMRTcell throughput, motivating the database approach

## Evidence

- [abstract] isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples, and extracting and visualizing various metrics.: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples, and extracting and visualizing various metrics."
- [methods] Python version and query capability.: "isoSeQL was developed using python 3.9.6"
- [readme] Origin story: PB.X.Y IDs cannot be matched across samples.: "I was working with multiple samples that I wanted to compare to each other, but quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched."
- [methods] UpSet plot dependency installed from GitHub.: "devtools::install_github("krassowski/complex-upset")"
