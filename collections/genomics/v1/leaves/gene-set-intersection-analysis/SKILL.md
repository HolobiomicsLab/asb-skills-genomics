---
name: gene-set-intersection-analysis
description: Use when the input is a disease-gene association table (e.g. GutUDB DISEASES
  module) and the goal is to identify which genes are shared across two or more intestinal
  diseases, or to tally per-disease gene counts.
when_to_use_negative:
- The goal is to compute total disease-gene association counts alone with no interest
  in shared genes — simple counting suffices and intersection is unnecessary.
- The input is raw multiomics profiles (e.g. GutUDB's 9 million generated profiles
  from eight omics data types) rather than curated disease-gene association records.
- The diseases of interest are not among GutUDB's 56 curated intestinal diseases,
  since intersection is only meaningful over curated gene sets.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: GutUDB
  role: Source database whose DISEASES module provides the curated disease-gene association
    records that are subset and intersected
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Python library used to parse association records, compute per-disease gene
    counts, and perform set intersection over gene sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:ef882e3496184a2fd7309bbd7af0b2c913b08bc80964b98f7777eed57ff5ca4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-set-intersection-analysis

## Summary

Computes the intersection of disease-associated gene sets across multiple intestine-related diseases curated in GutUDB's DISEASES module (e.g. colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease), yielding the shared common genes such as STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV that illustrate relationships between intestinal pathologies and genes.

## When to use

Use when the input is a disease-gene association table (e.g. GutUDB DISEASES module) and the goal is to identify which genes are shared across two or more intestinal diseases, or to tally per-disease gene counts.

## When NOT to use

- The goal is to compute total disease-gene association counts alone with no interest in shared genes — simple counting suffices and intersection is unnecessary.
- The input is raw multiomics profiles (e.g. GutUDB's 9 million generated profiles from eight omics data types) rather than curated disease-gene association records.
- The diseases of interest are not among GutUDB's 56 curated intestinal diseases, since intersection is only meaningful over curated gene sets.

## Inputs

- GutUDB DISEASES module disease-gene association data files from github.com/Ansanqi/GutUDB
- List of target intestine-related diseases (e.g. colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease)

## Outputs

- disease_gene_counts.csv containing per-disease and total gene-association counts
- shared_genes.txt containing the intersected common gene symbols

## How to apply

Load GutUDB's DISEASES module association records from the github.com/Ansanqi/GutUDB repository using Python/pandas. First parse the records and compute the total number of disease-gene associations, validating against the published total of 260,790. Subset the association table to the main intestine-related diseases and tally the number of gene associations per disease. Then use pandas set operations (e.g. functools.reduce over pd.Index.intersection) to compute the intersection of the gene sets across these diseases. Rationale: intersecting curated gene sets reveals genes shared across intestinal pathologies; the published validation anchor is the 260,790 total association count, and the expected shared-gene set includes STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV.

## Related tools

- **GutUDB** (Source database whose DISEASES module provides the curated disease-gene association records that are subset and intersected) — https://github.com/Ansanqi/GutUDB
- **pandas** (Python library used to parse association records, compute per-disease gene counts, and perform set intersection over gene sets)

## Examples

```
import pandas as pd; from functools import reduce; gene_sets = {d: set(g[g['disease']==d]['gene']) for d in diseases}; shared = reduce(set.intersection, gene_sets.values()); pd.Series(shared).to_csv('shared_genes.txt', index=False, header=False)
```

## Evaluation signals

- Computed total disease-gene association count equals the published figure of 260,790.
- disease_gene_counts.csv exists with one row per selected disease plus a total-count row.
- shared_genes.txt exists with one gene symbol per line and contains STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV.
- Each gene in shared_genes.txt is present in the association table for every selected intestine-related disease (intersection invariant).
- Per-disease association counts are non-negative integers consistent with the parsed association records.

## Limitations

- The shared-gene list depends on GutUDB's curation; the DISEASES module's 260,790 associations cover GutUDB's 56 intestinal diseases across six species, so non-intestinal diseases or diseases outside this curation are out of scope.
- The result reflects curated association records, not experimental verification of shared mechanism; genes like MEFV appear in the intersection but the biological interpretation (e.g. for constipation vs. cancer) is left to the analyst.

## Evidence

- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [other] The main intestine-related diseases in GutUDB share common genes such as STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV: [evidence span withheld — anchor status: not_found]
