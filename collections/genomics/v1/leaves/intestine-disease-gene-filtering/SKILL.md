---
name: intestine-disease-gene-filtering
description: Use when the input is GutUDB DISEASES-module association data and the
  goal is to tally gene associations per intestinal disease (colon cancer, colorectal
  cancer, IBD, constipation, diarrhea, etc.) and intersect gene sets across diseases.
when_to_use_negative:
- The analysis targets non-intestinal diseases outside the 56 intestinal diseases
  covered by GutUDB
- The input is raw multiomics profiles (e.g. the ~9 million omics profiles or THERAPY
  drug-disease interactions) rather than the DISEASES-module gene-association table
- A per-disease ranked gene list or p-value score is required — this skill produces
  counts and set intersections only, not statistical rankings
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3337
tools:
- name: GutUDB
  role: Source database whose DISEASES module provides the 260,790 disease-gene association
    records being tallied and intersected
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Python library used to parse association records, subset per disease, and
    compute gene-set intersections via set operations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:b308ce05c2e7f5bafb794b9ad4be5646ab667f45b314fd530e5edb9409c9ce28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# intestine-disease-gene-filtering

## Summary

A skill for computing per-disease gene-association counts and the set of genes shared across the main intestine-related diseases curated in GutUDB's DISEASES module, using pandas set operations over the 260,790 disease-gene association records.

## When to use

Use when the input is GutUDB DISEASES-module association data and the goal is to tally gene associations per intestinal disease (colon cancer, colorectal cancer, IBD, constipation, diarrhea, etc.) and intersect gene sets across diseases.

## When NOT to use

- The analysis targets non-intestinal diseases outside the 56 intestinal diseases covered by GutUDB
- The input is raw multiomics profiles (e.g. the ~9 million omics profiles or THERAPY drug-disease interactions) rather than the DISEASES-module gene-association table
- A per-disease ranked gene list or p-value score is required — this skill produces counts and set intersections only, not statistical rankings

## Inputs

- GutUDB DISEASES module disease-gene association data files from the Ansanqi/GutUDB repository
- List of target intestine-related diseases (e.g. colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease)

## Outputs

- disease_gene_counts.csv containing per-disease and total gene-association counts
- shared_genes.txt containing the gene symbols shared across the selected intestinal diseases

## How to apply

Load the DISEASES module data files from the GutUDB repository (github.com/Ansanqi/GutUDB) with Python/pandas and parse the association records, first computing the total disease-gene association count and validating it against the published figure of 260,790. Subset the association table to the main intestine-related diseases and tally the number of gene associations per disease. Compute the intersection of the per-disease gene sets using pandas set operations to obtain the shared common genes, which the source analysis reports as STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV. Write per-disease and total counts to disease_gene_counts.csv and the intersected gene symbols to shared_genes.txt, then verify both files exist with the expected row/line structure.

## Related tools

- **GutUDB** (Source database whose DISEASES module provides the 260,790 disease-gene association records being tallied and intersected) — https://github.com/Ansanqi/GutUDB
- **pandas** (Python library used to parse association records, subset per disease, and compute gene-set intersections via set operations)

## Examples

```
import pandas as pd; df = pd.read_csv('disease_gene_associations.tsv', sep='\t'); shared = sorted(set.intersection(*[set(df[df.disease == d].gene) for d in ['colon cancer', 'colorectal cancer', 'IBD']])); open('shared_genes.txt','w').write('\n'.join(shared))
```

## Evaluation signals

- Computed total disease-gene association count must exactly match the published figure of 260,790
- disease_gene_counts.csv exists with the expected row structure (one row per disease plus total)
- shared_genes.txt exists with one gene symbol per line matching the reported intersection (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV)
- Each gene in shared_genes.txt must be present in the gene set of every selected intestinal disease (set-intersection invariant)
- Manual expert review of the per-disease tallies against the GutUDB source data

## Limitations

- The shared-gene list depends on which diseases are selected as 'main intestine-related diseases'; a different disease subset changes the intersection
- Counts are only valid against the GutUDB DISEASES module snapshot; the repository data may be updated over time
- Manual expert review is the only listed validation tool, so automated checks beyond the 260,790-count invariant must be constructed by the practitioner

## Evidence

- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] GutUDB encompasses 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [other] Source card finding on shared genes across intestinal diseases: [evidence span withheld — anchor status: not_found]
