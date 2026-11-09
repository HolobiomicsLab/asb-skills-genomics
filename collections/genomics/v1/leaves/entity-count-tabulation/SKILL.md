---
name: entity-count-tabulation
description: Use when a paper or repository release reports aggregate entity counts
  (genes, profiles, diseases, species, tissue/cell lines) and you need to independently
  recompute those totals from the deposited data files to verify they match.
when_to_use_negative:
- You need fine-grained record-level content (e.g. individual disease-gene associations
  or specific drug-disease interactions) rather than headline totals — count tabulation
  only validates aggregate scale
- The release has no accompanying reported counts to compare against, so recomputation
  cannot serve as a validation exercise
- The data are spread across external sources not included in the release, making
  the recomputed counts incomparable to the article's totals
tools:
- name: Ansanqi/GutUDB
  role: Source repository whose downloaded data files are enumerated and counted to
    reproduce the aggregate statistics
  repo: https://github.com/Ansanqi/GutUDB
- name: GutUDB
  role: The multiomics database release whose entity totals (profiles, genes, tissue
    subtypes, cell lines, diseases, species) are tabulated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:70d6fb82b49a926a7bb4dc8c4e5abd381feb52de543ad54f5825bb84fcf795d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - Ansanqi/GutUDB
  - GutUDB
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# entity-count-tabulation

## Summary

Reproduces and validates the headline aggregate content statistics of a multiomics data release — in GutUDB's case the ~9 million profiles, 58,970 genes, 11 intestinal tissue subtypes, 63 intestinal cell lines, 56 intestinal diseases, and 6 species — by enumerating and counting distinct records in each category of the released data files against the values reported in the article.

## When to use

Use when a paper or repository release reports aggregate entity counts (genes, profiles, diseases, species, tissue/cell lines) and you need to independently recompute those totals from the deposited data files to verify they match.

## When NOT to use

- You need fine-grained record-level content (e.g. individual disease-gene associations or specific drug-disease interactions) rather than headline totals — count tabulation only validates aggregate scale
- The release has no accompanying reported counts to compare against, so recomputation cannot serve as a validation exercise
- The data are spread across external sources not included in the release, making the recomputed counts incomparable to the article's totals

## Inputs

- Data release files cloned from github.com/Ansanqi/GutUDB (omics profile files, gene annotation files, tissue/cell-line metadata, disease annotations, species metadata)
- Reported aggregate values from the article abstract for comparison

## Outputs

- gutudb_headline_stats.csv with columns: entity, recomputed count, reported value
- Per-category distinct-record counts (profiles, genes, tissue subtypes, cell lines, diseases, species)

## How to apply

Download the release from the project's GitHub repository (for GutUDB, github.com/Ansanqi/GutUDB) via git clone or the release archive. Enumerate the data files per entity category: omics profiles, gene annotations, tissue/cell-line metadata, disease annotations, and species metadata. Load each file with Python/pandas and count distinct records per entity, expecting roughly 9 million generated profiles from eight omics data types, 58,970 genes from 11 subtypes of intestinal tissue or 63 intestinal cell lines, 56 intestinal diseases, and 6 species. Assemble a statistics table with columns for entity, recomputed count, and reported value from the article abstract. Validate by checking exact matches for the discrete counts (58,970 genes; 11 tissue subtypes; 63 cell lines; 56 diseases; 6 species) and approximate agreement for the ~9 million profiles, then export the table (e.g. gutudb_headline_stats.csv).

## Related tools

- **Ansanqi/GutUDB** (Source repository whose downloaded data files are enumerated and counted to reproduce the aggregate statistics) — https://github.com/Ansanqi/GutUDB
- **GutUDB** (The multiomics database release whose entity totals (profiles, genes, tissue subtypes, cell lines, diseases, species) are tabulated)

## Examples

```
import pandas as pd; genes = pd.read_csv('genes.csv'); stats = {'genes': genes['gene'].nunique(), 'diseases': pd.read_csv('diseases.csv')['disease'].nunique()}
```

## Evaluation signals

- Recomputed gene count is exactly 58,970
- Recomputed counts are exactly 11 intestinal tissue subtypes, 63 intestinal cell lines, 56 intestinal diseases, and 6 species
- Total generated profiles agree approximately with the reported ~9 million (order-of-magnitude and eight omics data types)
- Output CSV has one row per entity with matching recomputed and reported values
- Distinct-record counts are computed with deduplication (e.g. pandas nunique), not raw row counts

## Limitations

- The ~9 million profile figure is approximate and eight-omics-aggregated, so only order-of-magnitude agreement can be expected
- Counts depend on the exact release snapshot; later updates to the repository will shift totals away from the reported values
- Aggregate tabulation cannot detect duplicate or mis-annotated records within a category — it validates scale, not correctness of individual entries

## Evidence

- [abstract] To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics"
- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
