---
name: aggregate-statistics-verification
description: Use when a data release (e.g. GutUDB from github.com/Ansanqi/GutUDB)
  must be validated by recomputing entity-level counts — ~9 million profiles, 58,970
  genes, 11 tissue subtypes, 63 cell lines, 56 diseases, 6 species — against reported
  abstract values.
when_to_use_negative:
- The goal is deep content analysis of GutUDB modules (e.g. the THERAPY module's 21,984
  drug-disease interactions or genomic variant content) rather than verifying headline
  release-scale counts
- No authoritative reported figures exist to compare against, so a recomputation alone
  cannot serve as a validation
edam_topics:
- http://edamontology.org/topic_3077
tools:
- name: Ansanqi/GutUDB
  role: Source repository to clone for the data release whose aggregate counts are
    recomputed and validated
  repo: github.com/Ansanqi/GutUDB
- name: GutUDB
  role: The multiomics database (approximately 9 million generated profiles from eight
    types of omics data sets) whose content statistics are being verified
  repo: github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:bca766f6340ae90b35665ca583e29a424e2a1bccd48d5d800476d2c1d85616e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Ansanqi/GutUDB
  - GutUDB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# aggregate-statistics-verification

## Summary

Recomputes the headline aggregate statistics of a curated data release (record counts per entity such as genes, diseases, species, and profiles) directly from the distributed data files and cross-checks them against the reported figures. This validates that a released database corpus matches the published overview before reuse.

## When to use

Use when a data release (e.g. GutUDB from github.com/Ansanqi/GutUDB) must be validated by recomputing entity-level counts — ~9 million profiles, 58,970 genes, 11 tissue subtypes, 63 cell lines, 56 diseases, 6 species — against reported abstract values.

## When NOT to use

- The goal is deep content analysis of GutUDB modules (e.g. the THERAPY module's 21,984 drug-disease interactions or genomic variant content) rather than verifying headline release-scale counts
- No authoritative reported figures exist to compare against, so a recomputation alone cannot serve as a validation

## Inputs

- GutUDB data release files from github.com/Ansanqi/GutUDB (omics profiles, gene annotations, tissue/cell-line metadata, disease annotations, species metadata)
- Reported aggregate values from the article abstract (e.g. 58,970 genes; 56 intestinal diseases; 260,790 disease-gene associations)

## Outputs

- Statistics table mapping entity → recomputed count → reported value (e.g. gutudb_headline_stats.csv)
- Pass/fail validation status per entity (exact match vs. approximate agreement for the ~9 million profiles)

## How to apply

Clone the release repository (github.com/Ansanqi/GutUDB via git clone or the GitHub release archive) and enumerate the data files holding omics profiles, gene annotations, tissue/cell-line metadata, disease annotations, and species metadata. Load each category with Python/pandas and count distinct records per entity, expecting exact matches to the reported aggregates: 58,970 genes, 11 subtypes of intestinal tissue, 63 intestinal cell lines, 56 intestinal diseases, 6 species, plus 260,790 disease-gene associations; the ~9 million generated profiles from eight types of omics data sets are checked for approximate agreement. Assemble a three-column table (entity, recomputed count, reported value) and save it, e.g. as gutudb_headline_stats.csv. Treat exact-count mismatches as release-integrity failures and investigate before using the data in downstream analyses.

## Related tools

- **Ansanqi/GutUDB** (Source repository to clone for the data release whose aggregate counts are recomputed and validated) — github.com/Ansanqi/GutUDB
- **GutUDB** (The multiomics database (approximately 9 million generated profiles from eight types of omics data sets) whose content statistics are being verified) — github.com/Ansanqi/GutUDB

## Examples

```
git clone https://github.com/Ansanqi/GutUDB && python -c "import pandas as pd; genes=pd.read_csv('GutUDB/genes.csv'); print(genes['gene'].nunique())"
```

## Evaluation signals

- Recomputed gene count exactly equals 58,970
- Recomputed intestinal tissue subtypes = 11, intestinal cell lines = 63, intestinal diseases = 56, and species = 6
- Disease-gene association count equals 260,790
- Generated-profile count agrees approximately with the reported ~9 million (order-of-magnitude match, since this aggregate is approximate)
- Output table gutudb_headline_stats.csv contains all entities with both recomputed and reported columns populated

## Limitations

- The ~9 million profiles figure is explicitly approximate in the article, so only order-of-magnitude agreement can be asserted for that entity; the other counts are exact-match checks
- Verification covers only the aggregate counts stated in the abstract/overview — it does not validate the correctness or completeness of individual records (e.g. the 72,248 SNPs, 54,131 CNVs, 1097 SVs, 92,888 mutated genes beyond their totals)

## Evidence

- [abstract] To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [other] Load each category of data file with Python/pandas and count the distinct records per entity: [evidence span withheld — anchor status: not_found]
