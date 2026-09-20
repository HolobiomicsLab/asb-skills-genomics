---
name: enrichment-network-visualization
description: Use when you have a set of common tncRNAs (tRF-5/tRF-3 series, 18-25
  nt) and need to identify their predicted mRNA targets and determine which biological
  processes are enriched among them.
when_to_use_negative:
- Input small RNAs are not tRF-5/tRF-3 series or fall outside the 18-25 nt range used
  to define common tncRNAs in this study
- The goal is tncRNA detection/annotation from sRNA-seq reads rather than target prediction
  and enrichment (use tncRNA Toolkit instead)
- Only differential expression of tncRNAs under stress is needed, with no target/enrichment
  interpretation
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0203
tools:
- name: psRNATarget
  role: Predicts protein-coding mRNA targets of the 18-25 nt common tncRNA queries
    against Arabidopsis cDNA and transposon sequences (2017 release, default parameters)
- name: g:Profiler
  role: Performs pathway enrichment analysis on the psRNATarget-predicted protein
    target set
- name: Cytoscape
  role: Visualization platform in which the g:Profiler enrichment results are explored
    as a network
- name: EnrichmentMap
  role: Cytoscape plugin used to render the enrichment results as an enrichment network
    of related terms
- name: AutoAnnotate
  role: Cytoscape plugin used to automatically annotate the enrichment network clusters
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:3bf128276fc8473bed8fc87be641230e8c1aed473c2cba4bcc567f8f6645e340
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - AutoAnnotate
  - Cytoscape
  - EnrichmentMap
  - g:Profiler
  - psRNATarget
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# enrichment-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict mRNA targets of conserved plant tncRNAs with psRNATarget, run pathway enrichment on the target protein set with g:Profiler, and visualize the enriched terms as a network in Cytoscape (EnrichmentMap + AutoAnnotate).

## When to use

Use when you have a set of common tncRNAs (tRF-5/tRF-3 series, 18-25 nt) and need to identify their predicted mRNA targets and determine which biological processes are enriched among them.

## When NOT to use

- Input small RNAs are not tRF-5/tRF-3 series or fall outside the 18-25 nt range used to define common tncRNAs in this study
- The goal is tncRNA detection/annotation from sRNA-seq reads rather than target prediction and enrichment (use tncRNA Toolkit instead)
- Only differential expression of tncRNAs under stress is needed, with no target/enrichment interpretation

## Inputs

- Common tncRNA query sequences (tRF-5 and tRF-3 series, 18-25 nt long)
- Arabidopsis thaliana cDNA sequences as probable mRNA targets
- Arabidopsis transposon sequences as probable targets

## Outputs

- psRNATarget predicted protein/mRNA target table
- g:Profiler pathway enrichment results
- Cytoscape enrichment network with EnrichmentMap clusters annotated via AutoAnnotate
- tncRNA_targets_enrichment.csv (combined target table + enrichment results)

## How to apply

Take the identified common tncRNAs (tRF-5 and tRF-3 series, 18-25 nt long) as query sequences and Arabidopsis cDNA and transposon sequences as probable targets in psRNATarget (2017 release) with default parameters. Collect the predicted protein targets from the psRNATarget output table and submit this target set to g:Profiler for pathway enrichment analysis. Import the g:Profiler enrichment results into Cytoscape and build the enrichment network with the EnrichmentMap plugin, then summarize/annotate term clusters using the AutoAnnotate plugin. Write the target table plus enrichment results to a single output file (tncRNA_targets_enrichment.csv) so targets and enrichment can be validated together. Validation: confirm the output contains a non-empty target table and enrichment results consistent with the paper's reported pattern that common tncRNA targets are majorly enriched during metabolic and developmental processes.

## Related tools

- **psRNATarget** (Predicts protein-coding mRNA targets of the 18-25 nt common tncRNA queries against Arabidopsis cDNA and transposon sequences (2017 release, default parameters))
- **g:Profiler** (Performs pathway enrichment analysis on the psRNATarget-predicted protein target set)
- **Cytoscape** (Visualization platform in which the g:Profiler enrichment results are explored as a network)
- **EnrichmentMap** (Cytoscape plugin used to render the enrichment results as an enrichment network of related terms)
- **AutoAnnotate** (Cytoscape plugin used to automatically annotate the enrichment network clusters)

## Evaluation signals

- Output file tncRNA_targets_enrichment.csv contains a non-empty target table (predicted protein targets present)
- The enrichment results section of the output is non-empty and includes pathway/biological-process terms
- Enriched terms are dominated by metabolic and developmental processes, matching the paper's reported pattern
- Query tncRNAs are restricted to the tRF-5 and tRF-3 series and are 18-25 nt in length
- The Cytoscape network renders enrichment clusters that AutoAnnotate can label without manual re-clustering errors

## Limitations

- Target predictions are in silico; the paper notes that DE tncRNAs and their putative targets require further validation before mechanisms can be assigned
- Only common tncRNAs (tRF-5/tRF-3 series, 18-25 nt) were queried against Arabidopsis cDNA and transposon sequences, so targets of other tncRNA subtypes (tRF-1, tRNA halves) are not covered by this workflow
- No single standardized nomenclature for tncRNA subtypes exists, which complicates reuse of the target-prediction input definition

## Evidence
