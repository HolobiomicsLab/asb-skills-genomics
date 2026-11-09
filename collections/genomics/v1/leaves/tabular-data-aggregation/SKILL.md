---
name: tabular-data-aggregation
description: Use when a tabular association table (disease-gene pairs from GutUDB's
  DISEASES module) must be tallied per category and intersected across groups to find
  shared genes such as STK11, CFTR, APC.
when_to_use_negative:
- The input is raw omics profiles (e.g., the ~9 million genomics/transcriptomics profiles)
  rather than curated association tables — other GutUDB modules apply
- The goal is drug-disease interaction analysis (21,984 THERAPY module interactions),
  not disease-gene association counting
- You need statistical association testing (p-values, enrichment) rather than deterministic
  set operations and counts
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: pandas
  role: Loads the DISEASES module files, parses association records, tallies per-disease
    counts, and computes gene-set intersections via set operations
- name: GutUDB
  role: Source of the 260,790 disease-gene association records and the curated list
    of 56 intestinal diseases
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:bdda8b38a897d22198056c49c77baa725180adb47e6f96e1a00928c8d6c0711b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tabular-data-aggregation

## Summary

Aggregate GutUDB's DISEASES module records into per-disease gene-association counts and a shared-gene intersection using pandas, reproducing the article's reported 260,790 disease-gene associations.

## When to use

Use when a tabular association table (disease-gene pairs from GutUDB's DISEASES module) must be tallied per category and intersected across groups to find shared genes such as STK11, CFTR, APC.

## When NOT to use

- The input is raw omics profiles (e.g., the ~9 million genomics/transcriptomics profiles) rather than curated association tables — other GutUDB modules apply
- The goal is drug-disease interaction analysis (21,984 THERAPY module interactions), not disease-gene association counting
- You need statistical association testing (p-values, enrichment) rather than deterministic set operations and counts

## Inputs

- GutUDB DISEASES module data files (disease-gene association records) from the Ansanqi/GutUDB repository
- List of target intestine-related disease labels (e.g., colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease)

## Outputs

- disease_gene_counts.csv (total and per-disease association counts)
- shared_genes.txt (one intersected gene symbol per line)

## How to apply

Load the GutUDB DISEASES module data files from the Ansanqi/GutUDB repository with Python/pandas. Parse the association records and compute the total number of disease-gene associations, validating it against the article's figure of 260,790. Subset the association table to the main intestine-related diseases (colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease, etc.), tally gene associations per disease, then compute the intersection of the gene sets with pandas set operations to obtain shared common genes (e.g., STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV). Decision point: if the computed total does not match 260,790, stop and inspect the parsing step rather than proceeding to the intersection.

## Related tools

- **pandas** (Loads the DISEASES module files, parses association records, tallies per-disease counts, and computes gene-set intersections via set operations)
- **GutUDB** (Source of the 260,790 disease-gene association records and the curated list of 56 intestinal diseases) — https://github.com/Ansanqi/GutUDB

## Examples

```
import pandas as pd; df = pd.read_csv('disease_gene_associations.tsv', sep='\t'); assert len(df) == 260790; shared = set.intersection(*[set(df[df.disease==d].gene) for d in diseases])
```

## Evaluation signals

- Computed total association count must equal the published figure of 260,790
- disease_gene_counts.csv exists with the expected row structure (one row per disease plus a total)
- shared_genes.txt exists with the expected line structure and contains known shared genes such as STK11, CFTR, APC, MLH1, MSH2
- Each gene listed as shared is present in the gene set of every target intestine-related disease (intersection invariant)
- Per-disease counts are consistent with the subsetted association table (sum of per-disease tallies reproducible from the same subset)

## Limitations

- Validation depends on exact agreement with the published total of 260,790; changes in GutUDB data versions will break the check
- The shared-gene list reflects only the subset of main intestine-related diseases chosen; other disease groupings yield different intersections
- Curated association counts capture relationships, not effect sizes or statistical significance

## Evidence

- [abstract] 260,790 disease-gene associations embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [other] Parse the association records and compute the total number of disease-gene associations, checking it against the reported total of 260,790.: [evidence span withheld — anchor status: not_found]
- [other] Compute the intersection of the gene sets across these diseases using pandas set operations to obtain the shared common genes.: [evidence span withheld — anchor status: not_found]
