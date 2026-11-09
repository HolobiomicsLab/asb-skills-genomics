---
name: gene-ontology-pathway-enrichment
description: Use when you have a set of small RNAs (e.g., common tRF-5/tRF-3 tncRNAs,
  18-25 nt) whose predicted protein-coding mRNA targets need pathway/GO enrichment
  analysis to infer biological function.
when_to_use_negative:
- Query small RNAs are not the common tRF-5/tRF-3 series in the 18-25 nt range (e.g.,
  tRNA halves or tRF-1s), since only common tncRNAs of that size class were used as
  query in this workflow
- The psRNATarget target table is empty (no predicted targets), so enrichment analysis
  has no meaningful gene set to test
- The species of interest is not supported by the chosen target/reference sequences
  and enrichment backends (e.g., non-Arabidopsis cDNA without a matching g:Profiler
  organism)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: psRNATarget
  role: Predicts protein-coding mRNA targets of the common tncRNAs (tRF-5/tRF-3, 18-25
    nt) using Arabidopsis cDNA and transposon sequences as probable targets (2017
    release, default parameters)
- name: g:Profiler
  role: Performs pathway enrichment analysis on the psRNATarget-derived protein target
    set
- name: Cytoscape
  role: Visualizes the enrichment results using the EnrichmentMap and AutoAnnotate
    plugins
- name: tncRNA Toolkit
  role: Upstream workflow that detects and classifies the tRF-5s, tRF-3s, tRF-1, and
    tRNA halves from which the common query tncRNAs are drawn
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:bbb283c2286462e1c7fb697f88178274257a9b0b0df8cf2694edbedcbfc321aa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Cytoscape
  - g:Profiler
  - psRNATarget
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-ontology-pathway-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functional enrichment of predicted mRNA targets of conserved plant tncRNAs (tRF-5 and tRF-3 series, 18-25 nt) using psRNATarget target prediction followed by g:Profiler pathway enrichment and Cytoscape visualization, revealing whether targets are enriched in metabolic and developmental processes.

## When to use

Use when you have a set of small RNAs (e.g., common tRF-5/tRF-3 tncRNAs, 18-25 nt) whose predicted protein-coding mRNA targets need pathway/GO enrichment analysis to infer biological function.

## When NOT to use

- Query small RNAs are not the common tRF-5/tRF-3 series in the 18-25 nt range (e.g., tRNA halves or tRF-1s), since only common tncRNAs of that size class were used as query in this workflow
- The psRNATarget target table is empty (no predicted targets), so enrichment analysis has no meaningful gene set to test
- The species of interest is not supported by the chosen target/reference sequences and enrichment backends (e.g., non-Arabidopsis cDNA without a matching g:Profiler organism)

## Inputs

- Common tncRNA query sequences (tRF-5 and tRF-3 series, 18-25 nt long)
- Arabidopsis cDNA sequences (probable mRNA targets)
- Arabidopsis transposon sequences (probable targets)
- Protein target list from psRNATarget output

## Outputs

- psRNATarget predicted protein target table
- g:Profiler pathway enrichment results
- Cytoscape EnrichmentMap/AutoAnnotate enrichment network
- tncRNA_targets_enrichment.csv (combined target table and enrichment results)

## How to apply

Use the common tncRNAs (tRF-5 and tRF-3 series; 18-25 nt long) as query sequences against Arabidopsis cDNA and transposon sequences in psRNATarget (2017 release) with default parameters to obtain the predicted protein targets. Submit the resulting protein target set to g:Profiler for pathway enrichment analysis. Visualize the enrichment results in Cytoscape using the EnrichmentMap and AutoAnnotate plugins to cluster related terms and annotate them. Write the target table plus enrichment results to tncRNA_targets_enrichment.csv. Validate that the output contains a non-empty target table and that the enrichment pattern (majorly metabolic and developmental processes) is consistent with the paper's reported result.

## Related tools

- **psRNATarget** (Predicts protein-coding mRNA targets of the common tncRNAs (tRF-5/tRF-3, 18-25 nt) using Arabidopsis cDNA and transposon sequences as probable targets (2017 release, default parameters))
- **g:Profiler** (Performs pathway enrichment analysis on the psRNATarget-derived protein target set)
- **Cytoscape** (Visualizes the enrichment results using the EnrichmentMap and AutoAnnotate plugins)
- **tncRNA Toolkit** (Upstream workflow that detects and classifies the tRF-5s, tRF-3s, tRF-1, and tRNA halves from which the common query tncRNAs are drawn) — https://github.com/skbinfo/tncRNA-Toolkit

## Evaluation signals

- Output tncRNA_targets_enrichment.csv contains a non-empty target table
- Enrichment results in the CSV are present and non-empty alongside the target table
- Enriched GO/pathway terms are majorly metabolic and developmental processes, matching the paper's reported pattern
- Only tRF-5 and tRF-3 series tncRNAs of 18-25 nt were used as query sequences
- Cytoscape EnrichmentMap renders clustered enrichment nodes with AutoAnnotate labels (visual sanity check)

## Limitations

- Predicted targets from psRNATarget are in-silico; the article notes further validation of DE tncRNAs and their putative targets is required to understand regulatory mechanisms
- Experimental validation and characterization are vital for distinguishing bona fide tncRNAs before downstream functional interpretation
- Whether AGOs associate with all tncRNAs is unresolved, soAGO-mediated targeting may not explain all predicted target regulation

## Evidence

- [methods] Common tncRNAs (tRF-5 and tRF-3 series; 18-25 nt long) were used as a query: [evidence span withheld — anchor status: not_found]
