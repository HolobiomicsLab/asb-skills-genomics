---
name: curated-database-data-validation
description: Use when reproducing or auditing published counts and relationship claims
  from a curated database download (e.g. THERAPY drug-disease interaction totals)
  or when checking that a specific entity (e.g. cisplatin) is well-connected in an
  integrated network layer.
when_to_use_negative:
- The input is raw experimental data rather than an already-curated database dump
  whose counts can be tallied and checked against published figures
- The analysis goal is novel statistical modeling or prediction rather than verification
  of published database statistics and network connectivity
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0602
tools:
- name: GutUDB
  role: Source curated database providing the THERAPY module drug-disease interactions
    and the disease-gene association layers being validated
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Loading and tallying the drug-disease interaction records and joining them
    with disease-gene associations
- name: networkx
  role: Extracting the colon-cancer subnetwork from the integrated drug-disease-gene
    graph and computing degree-based connectivity scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:a550e388d722215b002eb3c9d8f958b2472668f353414de4c157ce52d10463ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - 'GutUDB (GitHub: Ansanqi/GutUDB)'
  - networkx
  - pandas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# curated-database-data-validation

## Summary

Validates a curated multiomics database's reported statistics and internal consistency by programmatically re-deriving counts (e.g. GutUDB's 21,984 THERAPY drug-disease interactions across 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics) and confirming network connectivity of exemplar entries such as cisplatin in the drug-disease-gene network.

## When to use

Use when reproducing or auditing published counts and relationship claims from a curated database download (e.g. THERAPY drug-disease interaction totals) or when checking that a specific entity (e.g. cisplatin) is well-connected in an integrated network layer.

## When NOT to use

- The input is raw experimental data rather than an already-curated database dump whose counts can be tallied and checked against published figures
- The analysis goal is novel statistical modeling or prediction rather than verification of published database statistics and network connectivity

## Inputs

- THERAPY module drug-disease interaction records from the GutUDB GitHub repository (Ansanqi/GutUDB)
- GutUDB disease-gene association layer (260,790 associations from genomics/transcriptomics data)
- Reported reference values: 21,984 drug-disease interactions; 6,281 chemical compounds; 393 traditional medicinal herbs; 22 probiotics

## Outputs

- therapy_drug_disease_counts.csv with verified per-category interaction counts
- cisplatin_connectivity_score.csv with the exemplar drug's degree-based connectivity score in the colon-cancer subnetwork

## How to apply

Download the module-level data (e.g. the THERAPY module from the GutUDB GitHub repository Ansanqi/GutUDB) and load the drug-disease interaction records into pandas. Tally the total interactions and entries by therapeutic category, comparing each against the published reference values: 21,984 total drug-disease interactions, 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics. Join the drug-disease interactions with the 260,790 disease-gene associations from GutUDB's genomics/transcriptomics layers to construct an integrated drug-disease-gene graph, then use networkx to extract the disease-specific subnetwork (e.g. colon cancer). Compute degree-based connectivity scores for all drugs in that subnetwork, rank them, and confirm the exemplar drug (cisplatin) ranks among the top-connected entries. Persist the verified category counts and the exemplar's connectivity score as CSV outputs for the audit trail.

## Related tools

- **GutUDB** (Source curated database providing the THERAPY module drug-disease interactions and the disease-gene association layers being validated) — https://github.com/Ansanqi/GutUDB
- **pandas** (Loading and tallying the drug-disease interaction records and joining them with disease-gene associations)
- **networkx** (Extracting the colon-cancer subnetwork from the integrated drug-disease-gene graph and computing degree-based connectivity scores)

## Examples

```
import pandas as pd; therapy = pd.read_csv('therapy_drug_disease.csv'); counts = therapy.groupby('category').size(); assert counts['chemical compounds'] == 6281
```

## Evaluation signals

- Category counts match the reported values exactly: 21,984 total drug-disease interactions, 6,281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics
- The join with the disease-gene layer yields an integrated drug-disease-gene network without dropping interaction records
- Cisplatin ranks among the top-connected drugs in the colon-cancer subnetwork by degree-based connectivity score
- Both CSV outputs (therapy_drug_disease_counts.csv and cisplatin_connectivity_score.csv) are produced with schema-consistent rows

## Limitations

- Validation is anchored to figures published in the abstract; if the repository data are updated or versioned differently, exact-count checks may fail even when the database is correct
- Connectivity verification for cisplatin supports reliability of stored information but is a single-exemplar check, not a comprehensive curation audit

## Evidence

- [abstract] GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics: "GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics"
- [abstract] In total, 260,790 disease-gene associations are embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [other] Among the curated THERAPY associations, cisplatin (a chemotherapy drug for colon cancer patients) demonstrated strong connectivity in the drug-disease-gene network, supporting the reliability of the information stored in GutUDB.: [evidence span withheld — anchor status: not_found]
- [abstract] GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics"
