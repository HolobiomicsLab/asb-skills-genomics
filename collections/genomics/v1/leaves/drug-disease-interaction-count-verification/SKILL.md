---
name: drug-disease-interaction-count-verification
description: Use when auditing a therapy-module curation layer for internal consistency
  — e.g. recounting drug-disease associations by category against published totals,
  or confirming a known chemotherapy drug's network connectivity.
when_to_use_negative:
- The goal is to mine novel drug-disease associations or predict new therapeutics
  — this skill only recounts and validates existing GutUDB THERAPY curation, it does
  not infer new interactions.
- The input data are not GutUDB THERAPY-module records (e.g. a different drug-target
  or pharmacovigilance database), since the expected count totals 21,984 / 6,281 /
  393 / 22 are GutUDB-specific and would fail spuriously.
- The analysis targets a disease outside GutUDB's 56 intestinal diseases scope, where
  the disease-gene join and subnetwork extraction are not supported.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GutUDB
  role: Source database providing the THERAPY module drug-disease interactions and
    the 260,790 disease-gene associations joined into the drug-disease-gene network
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Loads and tallies the THERAPY drug-disease interaction records and performs
    the join with disease-gene associations
- name: networkx
  role: Constructs the integrated drug-disease-gene graph and extracts the colon-cancer
    subnetwork for degree-based drug connectivity scoring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:29fdea476df3603eda135c6b9363a51fef7c1b32e7dbc34fc6601d6cca5aaee1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - 'GutUDB (GitHub: Ansanqi/GutUDB)'
  - networkx
  - pandas
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# drug-disease-interaction-count-verification

## Summary

Verifies the curated drug-disease interaction counts in GutUDB's THERAPY module (21,984 total interactions across 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics) and validates database reliability by checking drug connectivity in an integrated drug-disease-gene network, using cisplatin in the colon-cancer subnetwork as the benchmark case.

## When to use

Use when auditing a therapy-module curation layer for internal consistency — e.g. recounting drug-disease associations by category against published totals, or confirming a known chemotherapy drug's network connectivity.

## When NOT to use

- The goal is to mine novel drug-disease associations or predict new therapeutics — this skill only recounts and validates existing GutUDB THERAPY curation, it does not infer new interactions.
- The input data are not GutUDB THERAPY-module records (e.g. a different drug-target or pharmacovigilance database), since the expected count totals 21,984 / 6,281 / 393 / 22 are GutUDB-specific and would fail spuriously.
- The analysis targets a disease outside GutUDB's 56 intestinal diseases scope, where the disease-gene join and subnetwork extraction are not supported.

## Inputs

- THERAPY module drug-disease interaction records from GutUDB (GitHub: Ansanqi/GutUDB), loadable with pandas
- GutUDB disease-gene association table (260,790 associations from the genomics/transcriptomics layers)
- Disease identifier for colon cancer to extract the disease-specific subnetwork

## Outputs

- therapy_drug_disease_counts.csv — verified category counts (21,984 / 6,281 / 393 / 22)
- cisplatin_connectivity_score.csv — cisplatin's degree-based connectivity score and rank in the colon-cancer drug-disease-gene subnetwork

## How to apply

Download the THERAPY module data from the GutUDB GitHub repository (Ansanqi/GutUDB) and load the drug-disease interaction records with pandas. Tally entries by therapeutic category and assert the totals equal the published values: 21,984 drug-disease interactions comprising 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics. Join these records with the 260,790 disease-gene associations from GutUDB's genomics/transcriptomics layers to construct an integrated drug-disease-gene graph using networkx. Extract the colon-cancer subnetwork, compute degree-based connectivity scores for all drugs, and rank them to confirm cisplatin (the reference chemotherapy drug for colon cancer patients) ranks among the top-connected drugs. The rationale is that a widely used chemotherapy agent showing strong connectivity in the disease subnetwork supports the reliability of the information stored in GutUDB.

## Related tools

- **GutUDB** (Source database providing the THERAPY module drug-disease interactions and the 260,790 disease-gene associations joined into the drug-disease-gene network) — https://github.com/Ansanqi/GutUDB
- **pandas** (Loads and tallies the THERAPY drug-disease interaction records and performs the join with disease-gene associations)
- **networkx** (Constructs the integrated drug-disease-gene graph and extracts the colon-cancer subnetwork for degree-based drug connectivity scoring)

## Evaluation signals

- Category counts table matches the reported values exactly: 21,984 total drug-disease interactions, 6,281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics.
- The drug-disease × disease-gene join yields a graph whose disease layer is consistent with GutUDB's 260,790 disease-gene associations.
- Cisplatin ranks among the top-connected drugs in the colon-cancer drug-disease-gene subnetwork by degree-based connectivity score.
- therapy_drug_disease_counts.csv contains one row per therapeutic category (chemical compounds, traditional medicinal herbs, probiotics) with counts summing to 21,984.
- cisplatin_connectivity_score.csv contains a single numeric connectivity score for cisplatin with a passing rank indicator.

## Limitations

- The connectivity check is a plausibility validation, not a biological discovery method — cisplatin passing the rank check supports reliability of stored information but does not quantify curation quality for other drugs or categories.
- Verification is dataset-version dependent: recounts will fail if the GutUDB THERAPY module is updated beyond the published 21,984 / 6,281 / 393 / 22 values.
- Degree-based connectivity in the drug-disease-gene network is biased toward highly pleiotropic drugs and heavily studied diseases like colon cancer.

## Evidence

- [abstract] THERAPY module curates 21,984 drug-disease interactions including 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics: "GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics"
- [abstract] 260,790 disease-gene associations embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [other] Workflow joins THERAPY records with disease-gene associations, extracts colon-cancer subnetwork, ranks drug connectivity: [evidence span withheld — anchor status: not_found]
