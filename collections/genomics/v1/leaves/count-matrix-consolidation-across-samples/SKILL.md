---
name: count-matrix-consolidation-across-samples
description: Use when you already hold a populated isoSeQL SQLite database (isoform,
  counts, isoform_ends, ends_counts, sampleData, exp tables) from SQANTI3-filtered
  classification files and need to compare isoform abundance or composition across
  samples.
when_to_use_negative:
- You have raw per-sample SQANTI3 classification files but no populated isoSeQL SQLite
  database — the phase-(i) database-building step must run first
- You are comparing only a single sample, since consolidation across samples with
  mismatched PB.X.Y IDs is the entire purpose
- Your isoform identifiers are already consistent across samples and you only need
  simple tabulation, not the database-backed consolidation machinery
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Source of pre-written phase-(ii) query functions against the SQLite database;
    also open to custom SQL queries
  repo: https://github.com/christine-liu/isoSeQL
- name: Python 3.9.6
  role: Runtime for pandas/numpy-based count-matrix consolidation and plotly structural-category
    bar plots
- name: pandas
  role: Querying and consolidating isoform records and aggregating counts by SQANTI3
    structural category
- name: numpy
  role: Numeric processing during isoform-by-sample count matrix generation
- name: plotly
  role: Rendering reads/isoforms-per-structural-category bar plots exported as PNG
- name: ComplexUpset
  role: Building cross-sample isoform intersection UpSet plots from per-sample isoform
    lists exported from SQLite
  repo: https://github.com/krassowski/complex-upset
- name: R 4.1.1
  role: Runtime for the ComplexUpset plotting step
- name: SQANTI3
  role: Upstream annotator whose structural categories and filtered classification
    outputs populate the database and define plot groupings
  repo: https://github.com/ConesaLab/SQANTI3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:60797f502d1b41d8502873e7c39ca2edeca582f3cf0cd6215cd01f672b06ae61
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# count-matrix-consolidation-across-samples

## Summary

isoSeQL's phase-(ii) database-querying skill: extract a consolidated isoform-by-sample count matrix (resolving mismatched PB.X.Y identifiers) from a populated SQLite database, and generate structural-category plots, cross-sample UpSet plots, and start/end read-distribution plots from it.

## When to use

Use when you already hold a populated isoSeQL SQLite database (isoform, counts, isoform_ends, ends_counts, sampleData, exp tables) from SQANTI3-filtered classification files and need to compare isoform abundance or composition across samples.

## When NOT to use

- You have raw per-sample SQANTI3 classification files but no populated isoSeQL SQLite database — the phase-(i) database-building step must run first
- You are comparing only a single sample, since consolidation across samples with mismatched PB.X.Y IDs is the entire purpose
- Your isoform identifiers are already consistent across samples and you only need simple tabulation, not the database-backed consolidation machinery

## Inputs

- Populated isoSeQL SQLite database built from SQANTI3-filtered classification files, with tables: isoform, counts, isoform_ends, ends_counts, sampleData, exp
- User-specified intron chain for start/end coordinate read-distribution queries

## Outputs

- Isoform-by-sample count matrix written to CSV
- Bar plots of reads and isoforms per SQANTI3 structural category (PNG, rendered with plotly)
- UpSet plots of cross-sample isoform intersections (PNG, rendered with ComplexUpset in R 4.1.1)
- Read-distribution plots across variable start/end coordinates for a queried intron chain (PNG)

## How to apply

Open the SQLite database built from SQANTI3-filtered classification files and inspect the isoform, counts, isoform_ends, ends_counts, sampleData, and exp tables. Using Python 3.9.6 with pandas and numpy, query and consolidate isoform records, resolving the PB.X.Y identifier mismatch that occurs when the same isoform receives different IDs in different experiments, and write the isoform-by-sample count matrix to CSV. Aggregate counts by SQANTI3 structural category with pandas and render reads/isoforms-per-category bar plots in plotly, exporting each as PNG. Export per-sample isoform lists from SQLite and build cross-sample intersection UpSet plots in R 4.1.1 with ComplexUpset; for a user-specified intron chain, pull start/end coordinate records from the isoform_ends and ends_counts tables and plot read distributions across variable ends. Pre-written query functions cover these tasks, but users can also write custom queries directly against the SQLite database.

## Related tools

- **isoSeQL** (Source of pre-written phase-(ii) query functions against the SQLite database; also open to custom SQL queries) — https://github.com/christine-liu/isoSeQL
- **Python 3.9.6** (Runtime for pandas/numpy-based count-matrix consolidation and plotly structural-category bar plots)
- **pandas** (Querying and consolidating isoform records and aggregating counts by SQANTI3 structural category)
- **numpy** (Numeric processing during isoform-by-sample count matrix generation)
- **plotly** (Rendering reads/isoforms-per-structural-category bar plots exported as PNG)
- **ComplexUpset** (Building cross-sample isoform intersection UpSet plots from per-sample isoform lists exported from SQLite) — https://github.com/krassowski/complex-upset
- **R 4.1.1** (Runtime for the ComplexUpset plotting step)
- **SQANTI3** (Upstream annotator whose structural categories and filtered classification outputs populate the database and define plot groupings) — https://github.com/ConesaLab/SQANTI3

## Examples

```
devtools::install_github("krassowski/complex-upset")
```

## Evaluation signals

- Every query function executes end-to-end on the example isoSeQL database without errors
- The isoform-by-sample count matrix CSV is produced and contains records from all loaded experiments, with PB.X.Y identifier collisions resolved across experiments
- Structural-category PNG bar plots (reads and isoforms per SQANTI3 category) are generated and populated with data
- UpSet PNG shows non-empty intersections among the per-sample isoform lists exported from SQLite
- Start/end read-distribution PNG for the queried intron chain is populated with records from the isoform_ends and ends_counts tables

## Limitations

- isoSeQL requires prior SQANTI3 (or SQANTI2) annotation; it is not a standalone isoform analysis tool
- The original 'mega sample' concatenation approach scales poorly with increased sample numbers and SMRTcell throughput, which motivated the database approach — very small comparisons may not need this machinery
- PB.X.Y identifiers differ between samples by construction, so any consolidation must rely on the database's identifier resolution rather than direct ID matching

## Evidence

- [abstract] Scope: extracting and visualizing metrics: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples, and extracting and visualizing various metrics."
- [intro] Identifier mismatch problem: "quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched"
- [methods] Python implementation version: "isoSeQL was developed using python 3.9.6"
- [methods] UpSet plotting dependency: "devtools::install_github("krassowski/complex-upset")"
- [readme] README positioning: "Database tool for comparing across many Iso-Seq runs analyzed through SQANTI3"
- [abstract] Repository availability: "isoSeQL is available on Github: https://github.com/christine-liu/isoSeQL"
