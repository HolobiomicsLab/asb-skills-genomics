---
name: gene-panel-membership-tallying
description: Use when a CPSR/PCGR virtual gene panel or superpanel gene-definition
  file must be decomposed into its contributing gene sources (CPG_SOURCE categories)
  and counted, e.g. to report panel composition as of a given data release date.
when_to_use_negative:
- The input is an annotated germline VCF to be classified and reported — use the CPSR
  report workflow, not panel-composition tallying.
- The panel gene definitions are already aggregated with per-source counts and a de-duplicated
  total, so no re-tallying is needed.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: CPSR
  role: Source of the virtual gene panel / superpanel gene-definition files and the
    CPG_SOURCE provenance categories being tallied
  repo: https://github.com/sigven/cpsr
- name: Python
  role: pandas-based parsing of the gene-definition file, construction of the gene
    x source membership matrix, and counting/de-duplication
- name: R
  role: Alternative implementation language for the tallying and CSV summary (CPSR
    itself is implemented in Python/R)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:defca146783c62a375b19f7ec522c70b2da89837eff1d1517b49234b2b5787a6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/sigven/cpsr
  repo_url: https://github.com/sigven/cpsr
  tools:
  - CPSR
  - Python
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-panel-membership-tallying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a de-duplicated gene-by-source membership matrix from CPSR's virtual-panel gene definitions and tally per-source counts (Panel App panels, COSMIC Cancer Gene Census, TCGA Pan-Cancer germline analysis, other user-contributed genes) to characterize panels such as the 433-gene exploratory superpanel.

## When to use

Use when a CPSR/PCGR virtual gene panel or superpanel gene-definition file must be decomposed into its contributing gene sources (CPG_SOURCE categories) and counted, e.g. to report panel composition as of a given data release date.

## When NOT to use

- The input is an annotated germline VCF to be classified and reported — use the CPSR report workflow, not panel-composition tallying.
- The panel gene definitions are already aggregated with per-source counts and a de-duplicated total, so no re-tallying is needed.

## Inputs

- CPSR superpanel gene-definition file (tabular, with per-gene source/provenance tags) from github.com/sigven/cpsr
- CPG_SOURCE annotation category definitions from the CPSR/PCGR reference bundle

## Outputs

- superpanel_source_composition.csv with one row per gene and binary membership flags per source
- Per-source gene count summary and de-duplicated union total (e.g. 433 genes as of June 2021)
- Source-membership records for validated genes of interest (e.g. BRCA1, BRCA2)

## How to apply

Load the superpanel gene-definition file shipped with the CPSR deposit (github.com/sigven/cpsr) using pandas, parsing each gene's provenance/source tags. Map the tags to the four contributing source categories (PanelApp-derived panels, COSMIC Cancer Gene Census, TCGA Pan-Cancer germline analysis, other curated/user-contributed), consistent with CPSR's CPG_SOURCE annotation categories. Build a binary membership matrix (gene x source), de-duplicate genes, and compute per-source counts plus the de-duplicated union. Verify BRCA1 and BRCA2 presence and record their source memberships. Write one row per gene to a CSV (e.g. superpanel_source_composition.csv) with a per-source count summary.

## Related tools

- **CPSR** (Source of the virtual gene panel / superpanel gene-definition files and the CPG_SOURCE provenance categories being tallied) — https://github.com/sigven/cpsr
- **Python** (pandas-based parsing of the gene-definition file, construction of the gene x source membership matrix, and counting/de-duplication)
- **R** (Alternative implementation language for the tallying and CSV summary (CPSR itself is implemented in Python/R))

## Examples

```
import pandas as pd; g = pd.read_csv('cpsr_superpanel_gene_definitions.tsv', sep='\t'); g[['symbol']].join(g['CPG_SOURCE'].str.get_dummies()).to_csv('superpanel_source_composition.csv', index=False); print(g['symbol'].nunique(), g['CPG_SOURCE'].value_counts().to_dict())
```

## Evaluation signals

- Per-source counts sum exactly to the de-duplicated 433-gene union (no duplicate or missing genes after de-duplication).
- Output CSV contains rows for BRCA1 and BRCA2 with non-empty source memberships.
- Every gene's provenance tags map to one or more of the four CPG_SOURCE categories; no unmapped tags remain.
- The membership matrix is strictly binary (0/1) with unique gene symbols as row keys.
- The reported total matches the documented superpanel size for the release date (433 genes as of June 2021).

## Limitations

- The gene count is release-specific: CPSR's panel content changes with data-bundle updates, so the 433-gene figure holds only as of June 2021 and must be re-verified per release.
- Genes can belong to multiple contributing sources, so naive per-source summation before de-duplication overcounts the union.
- CPSR offers more than 40 virtual expert-established panels; compositions tallied for one panel do not transfer to others.

## Evidence

- [other] CPSR's exploratory superpanel comprises 433 genes (as of June 2021) drawn from Panel App panels, COSMIC Cancer Gene Census, TCGA Pan-Cancer germline analysis, and other user-contributed genes.: [evidence span withheld — anchor status: not_found]
