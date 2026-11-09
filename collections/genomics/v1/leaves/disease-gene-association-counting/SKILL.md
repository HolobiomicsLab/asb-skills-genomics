---
name: disease-gene-association-counting
description: Use when a task requires reproducing or validating GutUDB DISEASES-module
  disease-gene association counts (target total 260,790) or computing gene-set intersections
  across the 56 curated intestinal diseases.
when_to_use_negative:
- The goal is counting drug-disease interactions from GutUDB's THERAPY module (21,984
  interactions incl. 6281 chemical compounds, 393 traditional medicinal herbs, 22
  probiotics) rather than disease-gene associations.
- The analysis targets omics profiles (e.g. the 137 RNA-seq data sets / 7.9 million
  transcriptional profiles, or the 72,248 SNPs / 54,131 CNVs) rather than the curated
  association tables.
- Input is an unrelated disease-gene database whose totals cannot be validated against
  GutUDB's published figure of 260,790.
tools:
- name: pandas
  role: Load the DISEASES module data files, tally associations per disease, and compute
    the gene-set intersection via set operations
- name: GutUDB
  role: Source database supplying the 260,790 disease-gene association records for
    56 intestinal diseases
  repo: github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:e9648de27d0abd9ed25e8c212f2e15b13f62cecbef41921f2c24e1a52e5259a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GutUDB
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# disease-gene-association-counting

## Summary

Reproduces the GutUDB DISEASES-module disease-gene association tallies — validating against the published total of 260,790 associations — and computes the intersection of gene sets across the main intestine-related diseases (colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease) to recover the shared gene list (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV).

## When to use

Use when a task requires reproducing or validating GutUDB DISEASES-module disease-gene association counts (target total 260,790) or computing gene-set intersections across the 56 curated intestinal diseases.

## When NOT to use

- The goal is counting drug-disease interactions from GutUDB's THERAPY module (21,984 interactions incl. 6281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics) rather than disease-gene associations.
- The analysis targets omics profiles (e.g. the 137 RNA-seq data sets / 7.9 million transcriptional profiles, or the 72,248 SNPs / 54,131 CNVs) rather than the curated association tables.
- Input is an unrelated disease-gene database whose totals cannot be validated against GutUDB's published figure of 260,790.

## Inputs

- GutUDB DISEASES module association data files (from github.com/Ansanqi/GutUDB)
- List of main intestine-related disease labels (colon cancer, colorectal cancer, constipation, diarrhea, inflammatory bowel disease, etc.)

## Outputs

- disease_gene_counts.csv (per-disease association counts plus total)
- shared_genes.txt (intersected gene symbols: STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV)

## How to apply

Obtain the GutUDB repository from github.com/Ansanqi/GutUDB and load the DISEASES module data files with Python/pandas. Parse the association records and compute the total number of disease-gene associations, checking it against the reported total of 260,790 before proceeding — this is the primary validation gate. Subset the association table to the main intestine-related diseases and tally the number of gene associations per disease. Compute the intersection of the per-disease gene sets using pandas set operations to obtain the shared common genes. Finally, write the per-disease and total counts to disease_gene_counts.csv and the intersected gene symbols to shared_genes.txt, since downstream reuse depends on these two files existing with the expected row/line structure.

## Related tools

- **pandas** (Load the DISEASES module data files, tally associations per disease, and compute the gene-set intersection via set operations)
- **GutUDB** (Source database supplying the 260,790 disease-gene association records for 56 intestinal diseases) — github.com/Ansanqi/GutUDB

## Examples

```
import pandas as pd; df = pd.read_csv('disease_gene_counts.csv'); shared = open('shared_genes.txt').read().split(); assert set(shared) == {'STK11','CFTR','BMPR1A','SMAD4','NOTCH1','PKD1','MLH1','MSH2','APC','MEFV'}
```

## Evaluation signals

- Computed total disease-gene association count must match the published figure of 260,790 exactly.
- disease_gene_counts.csv exists with one row per main intestine-related disease plus a total row of expected structure.
- shared_genes.txt exists and its lines contain exactly the expected shared gene symbols (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV).
- Per-disease counts are non-negative integers that sum consistently with the total association count.
- Manual expert review confirms the intersection covers the main intestine-related diseases curated in GutUDB.

## Limitations

- Validation relies on matching the single published total of 260,790 associations; partial or updated GutUDB snapshots may not match this figure.
- The shared-gene list reflects only the main intestine-related diseases subset selected in the workflow, not all 56 diseases across six species in GutUDB.
- The workflow relies on manual expert review rather than an automated verification harness.

## Evidence

- [abstract] 260,790 disease-gene associations are embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] 56 intestinal diseases across six species: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [other] Shared-gene intersection step: [evidence span withheld — anchor status: not_found]
