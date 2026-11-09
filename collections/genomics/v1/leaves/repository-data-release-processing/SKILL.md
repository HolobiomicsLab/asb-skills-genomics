---
name: repository-data-release-processing
description: Use when a published paper reports aggregate resource statistics (e.g.
  '58,970 genes from 11 tissue subtypes, 63 cell lines') and you must verify them
  by directly enumerating the data files in the accompanying repository release.
when_to_use_negative:
- Input is already a summarized statistics table — no enumeration of the release files
  is needed
- The goal is to analyze the biological content of the omics data (e.g. differential
  expression across the 137 RNA-seq data sets) rather than to audit release completeness
- The repository provides no structured data files matching the reported entities,
  so counts cannot be recomputed independently
edam_operation: http://edamontology.org/operation_2409
edam_topics: []
tools:
- name: Ansanqi/GutUDB
  role: Source repository whose release archive is cloned and enumerated to recompute
    entity counts
  repo: https://github.com/Ansanqi/GutUDB
- name: GutUDB
  role: The multiomics database itself; its data release supplies the profiles, gene,
    disease, and species records being counted
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Used to load each category of data file and count distinct records per entity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:f8bc01a72a2e9fe252ea44b987ef8bc0e2d9234365069ea966782cfa7640183a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - Ansanqi/GutUDB
  - GutUDB
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# repository-data-release-processing

## Summary

Enumerate and recompute headline entity counts (profiles, genes, tissue subtypes, cell lines, diseases, species) from a published multiomics data release, then reconcile recomputed values against the reported aggregates to validate the release. Applied here to GutUDB, which bundles ~9 million omics profiles, 58,970 genes, 11 intestinal tissue subtypes, 63 cell lines, 56 diseases, and 6 species.

## When to use

Use when a published paper reports aggregate resource statistics (e.g. '58,970 genes from 11 tissue subtypes, 63 cell lines') and you must verify them by directly enumerating the data files in the accompanying repository release.

## When NOT to use

- Input is already a summarized statistics table — no enumeration of the release files is needed
- The goal is to analyze the biological content of the omics data (e.g. differential expression across the 137 RNA-seq data sets) rather than to audit release completeness
- The repository provides no structured data files matching the reported entities, so counts cannot be recomputed independently

## Inputs

- GitHub data release or clone of Ansanqi/GutUDB (gene annotations, tissue/cell-line metadata, disease annotations, species metadata, omics profile files)
- Reported aggregate statistics from the article abstract (e.g. 58,970 genes; 11 tissue subtypes; 63 cell lines; 56 diseases; 6 species; ~9 million profiles)

## Outputs

- Per-entity count table mapping entity -> recomputed count -> reported value (e.g. gutudb_headline_stats.csv)
- Pass/fail reconciliation report per headline statistic (exact match for discrete counts, approximate for profile volumes)

## How to apply

Clone the repository release (e.g. github.com/Ansanqi/GutUDB via git clone or the GitHub release archive) and enumerate the data files by category: omics profiles, gene annotations, tissue/cell-line metadata, disease annotations, and species metadata. Load each category with Python/pandas and count distinct records per entity type rather than raw rows, since duplicates across omics modalities (RNA-seq transcriptional profiles, SNPs, CNVs, SVs, mutated genes) are expected. Assemble a statistics table with columns for entity, recomputed count, and the value reported in the article abstract. Treat the headline aggregates as validation targets: exact match for discrete annotations (58,970 genes, 11 intestinal tissue subtypes, 63 cell lines, 56 diseases, 6 species, 260,790 disease-gene associations) and approximate agreement for volume claims (approximately 9 million generated profiles from eight omics types). Save the reconciliation table (e.g. gutudb_headline_stats.csv) so mismatches are auditable.

## Related tools

- **Ansanqi/GutUDB** (Source repository whose release archive is cloned and enumerated to recompute entity counts) — https://github.com/Ansanqi/GutUDB
- **GutUDB** (The multiomics database itself; its data release supplies the profiles, gene, disease, and species records being counted) — https://github.com/Ansanqi/GutUDB
- **pandas** (Used to load each category of data file and count distinct records per entity)

## Examples

```
python -c "import pandas as pd,glob; stats={f:pd.read_csv(f).nunique().to_dict() for f in glob.glob('GutUDB/**/*.csv',recursive=True)}; print(stats)"
```

## Evaluation signals

- Recomputed gene count exactly equals 58,970
- Recomputed counts of 11 intestinal tissue subtypes, 63 intestinal cell lines, 56 intestinal diseases, and 6 species each exactly match the reported aggregates
- Total generated profiles approximately agree with the reported ~9 million figure from eight omics types
- Recomputed disease-gene association count reconciles with the reported 260,790
- Output table has the required columns (entity, recomputed count, reported value) and is saved as gutudb_headline_stats.csv

## Limitations

- The ~9 million profile figure is an approximate/accumulated volume, so only approximate agreement, not exact match, is achievable for that entity
- Counts reflect the release snapshot at publication time; later repository updates may shift discrete counts away from the reported values
- Abstract-level aggregates are only reported for the entity classes listed (genes, tissue subtypes, cell lines, diseases, species, associations); other content (e.g. 21,984 drug-disease interactions, 6281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics) must be validated separately

## Evidence

- [abstract] GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] GutUDB incorporated 58,970 genes from 11 subtypes of intestinal tissue or 63 intestinal cell lines: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics"
- [abstract] 260,790 disease-gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
