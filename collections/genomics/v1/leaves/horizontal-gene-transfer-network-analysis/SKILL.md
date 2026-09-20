---
name: horizontal-gene-transfer-network-analysis
description: Use when you have per-sample, species-resolved ARG abundance tables (cpg)
  from long-read metagenomes and need to assess ARG co-occurrence among carriers (e.g.,
  sul/mcr profiles) to flag candidate horizontal transfer across countries or host
  groups.
when_to_use_negative:
- Input is short-read data — Argo's approach depends on long-read overlap graph clustering
  (Nanopore/PacBio), so correlation-of-carriers analysis from short reads is unsupported
- You need de novo genomic context of ARGs (plasmid/chromosome localization, novel
  syntenies) — the article states assembly-based (metaFlye + Kraken2) or binning-based
  (SemiBin2 + GTDB-Tk) methods are better suited for that
- Carriers are extremely rare (< 1x genome copy) — Argo marks such species 'unclassified',
  so they would be missing from the network
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Argo
  role: Produces the species-resolved ARG-host abundance table (types/subtypes, hosts,
    cpg) whose sul/mcr carrier profiles feed the Spearman correlation network
  repo: https://github.com/xinehc/argo
- name: R (ggplot2)
  role: Computes Spearman rank correlations over sul/mcr carrier profiles, omits edges
    < 0.1, and renders the co-occurrence network
- name: Melon (with Kraken2 PlusPF pre-filtering)
  role: Estimates prokaryotic genome copies so ARG abundances are normalized to cpg
    before correlation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:c5a7f389f013a2bafa8a542b269ea314b5b70a0ee45e131d8b59bc5de8647f06
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Kraken2
  - Melon
  - Melon (with Kraken2 PlusPF pre-filtering)
  - Porechop
  - R
  - R (ggplot2)
  - ggplot2
  - nanoq
  - vegan
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# horizontal-gene-transfer-network-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers putative ARG movement between bacterial hosts by building a Spearman rank correlation network over species-resolved ARG carrier abundances (cpg) produced by Argo, focusing on sul and mcr subtypes in E. coli across human groups and non-human primates.

## When to use

Use when you have per-sample, species-resolved ARG abundance tables (cpg) from long-read metagenomes and need to assess ARG co-occurrence among carriers (e.g., sul/mcr profiles) to flag candidate horizontal transfer across countries or host groups.

## When NOT to use

- Input is short-read data — Argo's approach depends on long-read overlap graph clustering (Nanopore/PacBio), so correlation-of-carriers analysis from short reads is unsupported
- You need de novo genomic context of ARGs (plasmid/chromosome localization, novel syntenies) — the article states assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are better suited for that
- Carriers are extremely rare (< 1x genome copy) — Argo marks such species 'unclassified', so they would be missing from the network

## Inputs

- Quality-controlled long-read metagenomic FASTQ samples (Nanopore/PacBio fecal samples, 317 human + 12 chimpanzee/bonobo)
- Argo + SARG+ per-sample output tables listing ARG types/subtypes, predicted host species, and abundances in copies per genome (cpg)
- E. coli type-level ARG abundance matrix per country group (CN, KR, SG, DE, US, NH)
- Sul and mcr subtype abundance profiles extracted per carrier

## Outputs

- Spearman rank correlation network (edge list) of sul/mcr ARG carriers with edges < 0.1 omitted
- ggplot2 network visualization of sul/mcr co-occurrence across host species and country groups
- Group-level sul/mcr presence/absence summary (e.g., all sul/mcr genes absent from NH primates)

## How to apply

First generate species-resolved ARG abundances by running Argo with the SARG+ database on quality-controlled long reads (Porechop --discard_middle, then nanoq --min-qual 10 --min-len 1000); Argo assigns hosts to ARG-carrying read clusters via MCL graph clustering (inflation=2, expansion=2) and greedy weighted set cover, estimating genome copies with Melon. From the Argo output, aggregate E. coli type-level ARG abundances (cpg) per country group (CN, KR, SG, DE, US, NH) and extract the sul and mcr subtype profiles per carrier. Compute pairwise Spearman rank correlations across these carrier profiles in R and build the co-occurrence network, omitting edges with correlation < 0.1. Visualize the network with ggplot2 alongside group-level comparisons (Kruskal–Wallis plus Bonferroni-corrected Mann–Whitney tests). The rationale for cluster-level host labeling is that ARGs are prone to horizontal gene transfer and appear in multiple genetic locations, so per-read taxonomic classifiers are error-prone; Argo's read-cluster evidence yields carrier assignments stable enough for correlation analysis.

## Related tools

- **Argo** (Produces the species-resolved ARG-host abundance table (types/subtypes, hosts, cpg) whose sul/mcr carrier profiles feed the Spearman correlation network) — https://github.com/xinehc/argo
- **R (ggplot2)** (Computes Spearman rank correlations over sul/mcr carrier profiles, omits edges < 0.1, and renders the co-occurrence network)
- **Melon (with Kraken2 PlusPF pre-filtering)** (Estimates prokaryotic genome copies so ARG abundances are normalized to cpg before correlation)

## Examples

```
R: rho <- cor(t(sul_mcr_profiles), method = "spearman"); rho[rho < 0.1] <- 0
```

## Evaluation signals

- Every retained network edge has Spearman correlation >= 0.1 (edges < 0.1 must be omitted)
- Node set restricted to sul/mcr subtype carriers extracted from E. coli type-level cpg tables per country
- Reproduction anchor: E. coli sul2 at 0.705 cpg in the CN group
- Validation anchor: all sul/mcr genes absent from NH (non-human primate) samples in the network input profiles
- Input abundances are cpg-normalized (Argo/Melon genome-copy estimates), not raw read counts

## Limitations

- Correlation edges are indirect co-occurrence evidence — Argo assigns hosts but not ARG genomic locations (plasmid vs chromosome), so edges do not prove shared mobile-element transfer
- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- By default, Argo considers ARGs of species with less than one genome copy (1x) as 'unclassified', so rare carriers are excluded from the network
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the SARG+ database

## Evidence
