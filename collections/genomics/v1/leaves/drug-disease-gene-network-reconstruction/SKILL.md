---
name: drug-disease-gene-network-reconstruction
description: 'Use when reproducing or extending GutUDB THERAPY-module analyses: counting
  curated drug-disease interactions by category, or assessing whether a specific drug
  (e.g., cisplatin for colon cancer) shows strong connectivity in an integrated drug-disease-gene
  graph.'
when_to_use_negative:
- The goal is only to browse or download GutUDB records via its Browse/Query/Download
  functionalities without constructing or scoring an integrated network.
- The analysis targets non-THERAPY GutUDB content (e.g., the 72,248 SNPs, 54,131 CNVs,
  1097 SVs, 92,888 mutated genes, or RNA-seq/m6A layers) rather than drug-disease
  interactions and disease-gene associations.
- Input data are not the GutUDB THERAPY module and disease-gene layers, so the 21,984/6,281/393/22
  validation counts and the cisplatin connectivity check cannot be meaningfully applied.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: GutUDB
  role: Source database whose THERAPY module (drug-disease interactions) and genomics/transcriptomics
    layers (disease-gene associations) are downloaded and integrated into the network
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Loads the THERAPY drug-disease interaction records and performs the category
    tallies and the join with disease-gene associations
- name: networkx
  role: Builds the integrated drug-disease-gene graph, extracts the colon-cancer subnetwork,
    and computes degree-based connectivity scores for drug ranking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:261644099483bdccf61a76cdf809eabd87f2992d95eb425e4874ad6a6b0bee29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - 'GutUDB (GitHub: Ansanqi/GutUDB)'
  - networkx
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# drug-disease-gene-network-reconstruction

## Summary

Reconstructs and validates the integrated drug-disease-gene network from GutUDB's THERAPY module (21,984 drug-disease interactions spanning 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics) joined with GutUDB's 260,790 disease-gene associations, then quantifies per-drug connectivity (e.g., cisplatin in the colon-cancer subnetwork) via networkx degree-based ranking.

## When to use

Use when reproducing or extending GutUDB THERAPY-module analyses: counting curated drug-disease interactions by category, or assessing whether a specific drug (e.g., cisplatin for colon cancer) shows strong connectivity in an integrated drug-disease-gene graph.

## When NOT to use

- The goal is only to browse or download GutUDB records via its Browse/Query/Download functionalities without constructing or scoring an integrated network.
- The analysis targets non-THERAPY GutUDB content (e.g., the 72,248 SNPs, 54,131 CNVs, 1097 SVs, 92,888 mutated genes, or RNA-seq/m6A layers) rather than drug-disease interactions and disease-gene associations.
- Input data are not the GutUDB THERAPY module and disease-gene layers, so the 21,984/6,281/393/22 validation counts and the cisplatin connectivity check cannot be meaningfully applied.

## Inputs

- GutUDB THERAPY module drug-disease interaction records (from GitHub repo Ansanqi/GutUDB)
- GutUDB disease-gene association layer (260,790 associations from genomics/transcriptomics data)
- Disease of interest identifier (e.g., colon cancer) for subnetwork extraction

## Outputs

- therapy_drug_disease_counts.csv — per-category counts of drug-disease interactions (chemical compounds, traditional medicinal herbs, probiotics)
- cisplatin_connectivity_score.csv — degree-based connectivity score and rank for cisplatin in the colon-cancer subnetwork
- Integrated drug-disease-gene network (networkx graph) and ranked per-drug degree table for the disease subnetwork

## How to apply

Download the THERAPY module data from the GutUDB GitHub repository (Ansanqi/GutUDB) and load the drug-disease interaction records with pandas; tally entries by therapeutic category and verify totals against the reported 21,984 interactions, 6,281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics. Join the drug-disease interactions with the 260,790 disease-gene associations from GutUDB's genomics/transcriptomics layers to construct the integrated drug-disease-gene network. Extract the disease-specific subnetwork (e.g., colon cancer) with networkx, compute degree-based connectivity scores for all drugs in that subnetwork, and rank drugs so target compounds like cisplatin can be located within the ranking. The rationale is that strong connectivity of a known chemotherapy drug for colon cancer patients supports the reliability of the information stored in GutUDB.

## Related tools

- **GutUDB** (Source database whose THERAPY module (drug-disease interactions) and genomics/transcriptomics layers (disease-gene associations) are downloaded and integrated into the network) — https://github.com/Ansanqi/GutUDB
- **pandas** (Loads the THERAPY drug-disease interaction records and performs the category tallies and the join with disease-gene associations)
- **networkx** (Builds the integrated drug-disease-gene graph, extracts the colon-cancer subnetwork, and computes degree-based connectivity scores for drug ranking)

## Examples

```
import pandas as pd; therapy = pd.read_csv('GutUDB_THERAPY_drug_disease.csv'); counts = therapy.groupby('category').size()
```

## Evaluation signals

- Category counts must match the reported values exactly: 21,984 total drug-disease interactions, 6,281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics.
- The disease-gene layer joined into the network must contain 260,790 disease-gene associations.
- Cisplatin must rank among the top-connected drugs in the colon-cancer drug-disease-gene subnetwork for the connectivity check to pass.
- therapy_drug_disease_counts.csv must contain one row per therapeutic category with the verified counts; cisplatin_connectivity_score.csv must contain a single degree-based score for cisplatin.
- The integrated network must contain nodes of all three types (drug, disease, gene), confirming the join was applied rather than a drug-only or gene-only graph.

## Limitations

- Validation totals are exact-match (21,984 / 6,281 / 393 / 22); any upstream change to the GutUDB THERAPY module data will break the counts check.
- Connectivity is degree-based within a single disease subnetwork (colon cancer), so it reflects network centrality, not experimentally validated therapeutic efficacy.
- The reliability interpretation rests on cisplatin being a known chemotherapy drug for colon cancer patients; connectivity ranking alone is a supporting, not definitive, signal.

## Evidence

- [abstract] GutUDB's THERAPY module curates 21,984 drug-disease interactions including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics: "GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics"
- [abstract] Disease-gene association layer size used in the network join: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] Drug connectivity finding supporting reliability of the network: "identified various potential clinical therapeutics"
