---
name: multiomics-database-querying
description: Use when the task requires retrieving or reproducing counts, gene-level
  omics profiles (e.g., METTL3 CNV deletion frequency or CNV-vs-RNA-expression correlation),
  or disease-gene/drug-disease statistics from the GutUDB release.
when_to_use_negative:
- The question concerns an intestinal-disease omics resource other than GutUDB — its
  counts (58,970 genes, 56 diseases, 6 species) are specific to this release.
- You need raw sequencing data or novel analysis beyond the pre-computed profiles
  GutUDB ships (e.g., de novo variant calling from FASTQs).
- The analysis targets non-intestinal tissues/diseases outside GutUDB's 56 intestinal
  diseases and 6 species scope.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: Ansanqi/GutUDB
  role: Source repository hosting the GutUDB data release (omics profile tables and
    metadata) that is cloned and enumerated with Python/pandas
  repo: https://github.com/Ansanqi/GutUDB
- name: GutUDB
  role: The database being queried; provides Browse, Query, Visualization, and Download
    functionality over its omics data sets
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:5ae493008ea2f559f238d4b20e957773af52fd5de365654f57f498f44ed50bdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - Ansanqi/GutUDB
  - GutUDB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multiomics-database-querying

## Summary

Query and reproduce aggregate statistics and gene-level analyses from GutUDB, a multiomics database for intestinal diseases that bundles ~9 million profiles across eight omics types (genomics, RNA-seq transcriptomics, m6A, microbe-metabolite relationships) with 58,970 genes and 260,790 disease-gene associations.

## When to use

Use when the task requires retrieving or reproducing counts, gene-level omics profiles (e.g., METTL3 CNV deletion frequency or CNV-vs-RNA-expression correlation), or disease-gene/drug-disease statistics from the GutUDB release.

## When NOT to use

- The question concerns an intestinal-disease omics resource other than GutUDB — its counts (58,970 genes, 56 diseases, 6 species) are specific to this release.
- You need raw sequencing data or novel analysis beyond the pre-computed profiles GutUDB ships (e.g., de novo variant calling from FASTQs).
- The analysis targets non-intestinal tissues/diseases outside GutUDB's 56 intestinal diseases and 6 species scope.

## Inputs

- GutUDB data release files from github.com/Ansanqi/GutUDB (omics profile tables, gene annotations, tissue/cell-line metadata, disease and species annotations)
- METTL3-focused CNV genomics profiles for colorectal cancer
- Matching METTL3 RNA-seq transcriptomics profiles

## Outputs

- gutudb_headline_stats.csv (entity, recomputed count, reported value)
- mettl3_cnv_results.csv (CNV deletion frequency, CNV-vs-RNA-expression correlation coefficient)
- Recomputed aggregate statistics table per omics entity

## How to apply

Clone the GutUDB release from github.com/Ansanqi/GutUDB and enumerate the data files per omics category (genomics: 72,248 SNPs, 54,131 CNVs, 1097 SVs, 92,888 mutated genes; transcriptomics: 137 RNA-seq data sets with 7.9 million profiles; 1908 m6A sites; 2764 gut microbe-metabolite relationships). Load each file with Python/pandas and either (a) recount distinct records per entity and diff against the reported aggregates (58,970 genes, 11 intestinal tissue subtypes, 63 cell lines, 56 diseases, 6 species, ~9 million profiles), or (b) subset the files for a target gene (e.g., METTL3) in colorectal cancer, computing CNV deletion frequency as the fraction of samples with a deletion call (expected 0.04) and Pearson/Spearman correlation between CNV state and RNA expression to confirm a weak association. Assemble results into small CSV tables (e.g., gutudb_headline_stats.csv, mettl3_cnv_results.csv) with recomputed vs. reported values for validation.

## Related tools

- **Ansanqi/GutUDB** (Source repository hosting the GutUDB data release (omics profile tables and metadata) that is cloned and enumerated with Python/pandas) — https://github.com/Ansanqi/GutUDB
- **GutUDB** (The database being queried; provides Browse, Query, Visualization, and Download functionality over its omics data sets) — https://github.com/Ansanqi/GutUDB

## Examples

```
git clone https://github.com/Ansanqi/GutUDB && python -c "import pandas as pd; df=pd.read_csv('mettl3_cnv.csv'); freq=(df['cnv']=='del').mean(); print(freq)"
```

## Evaluation signals

- Exact match of recomputed gene count to the reported 58,970 genes, 11 tissue subtypes, 63 cell lines, 56 diseases, and 6 species
- Approximate agreement of recomputed profile total with the reported ~9 million profiles from eight omics types
- METTL3 CNV deletion frequency in CRC reproduces the reported value of 0.04
- METTL3 CNV-vs-RNA-expression correlation (Pearson or Spearman) falls in the weak range despite significantly high RNA expression
- Output CSVs contain both recomputed and reported values, enabling a direct column-wise diff against the article's abstract figures

## Limitations

- Headline profile counts are approximate ('approximately 9 million'), so only approximate agreement is expected for that metric; exact matching is only valid for the enumerated counts (58,970 genes, 11 tissue subtypes, 63 cell lines, 56 diseases, 6 species).
- The METTL3 result shows CNV deletion frequency of only 0.04 and a weak CNV-expression correlation, so CNV-based queries for genes with low CNV burden may yield unstable or uninformative signals.

## Evidence

- [abstract] approximately 9 million profiles across eight omics types: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] 58,970 genes from 11 tissue subtypes or 63 cell lines: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics"
- [abstract] genomics content counts: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes"
- [abstract] four core functionalities: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] 260,790 disease-gene associations: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
