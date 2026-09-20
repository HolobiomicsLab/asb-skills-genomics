---
name: long-read-overlap-graph-construction
description: Use when quality-controlled ONT/PacBio reads flagged as ARG-containing
  need to be grouped into clusters (shared genomic origin) before species-level ARG
  host assignment, e.g., inside the Argo workflow.
when_to_use_negative:
- Input reads are short second-generation reads, where overlaps are too sparse/uninformative
  and per-read classifiers like Kraken2 or Centrifuge are the alternative strategies
- Coverage is insufficient (below ~1x), where inadequate read overlaps make clustering
  ineffective and taxonomy unstable
- The goal is de novo identification of ARG genomic contexts in unstudied environments,
  where assembly-based (metaFlye) or binning-based (SemiBin2 + GTDB-Tk) approaches
  are better suited
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Argo
  role: End-to-end profiler that runs the overlap-graph clustering and downstream
    set-cover species assignment
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: All-vs-all approximate read overlapping ('-x ava-ont') producing overlaps
    with dv divergence tags
- name: MCL
  role: Graph clustering (expansion e = 2, inflation r = 2, pruning z < 1/n) that
    decomposes the overlap graph into read clusters
- name: DIAMOND
  role: Aligns reads against indexed SARG+ proteins to identify the ARG-containing
    reads that enter the overlap graph
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:c6da8557024435c28e6a4f2ac96acdf143520907ea7ede1defefb2693a02ef5b
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
  - DIAMOND
  - MCL
  - Melon
  - minimap2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# long-read-overlap-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a read-overlap graph from ARG-containing long reads using minimap2 all-vs-all approximate mapping with divergence (dv) tags, then decomposes it into read clusters with MCL so that Argo can assign species labels at the cluster rather than read level.

## When to use

Use when quality-controlled ONT/PacBio reads flagged as ARG-containing need to be grouped into clusters (shared genomic origin) before species-level ARG host assignment, e.g., inside the Argo workflow.

## When NOT to use

- Input reads are short second-generation reads, where overlaps are too sparse/uninformative and per-read classifiers like Kraken2 or Centrifuge are the alternative strategies
- Coverage is insufficient (below ~1x), where inadequate read overlaps make clustering ineffective and taxonomy unstable
- The goal is de novo identification of ARG genomic contexts in unstudied environments, where assembly-based (metaFlye) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited

## Inputs

- Quality-controlled long reads (Nanopore or PacBio, e.g., FASTA such as example.fa.gz)
- Set of ARG-containing reads selected after DIAMOND alignment against the indexed SARG+ protein database

## Outputs

- Filtered overlap set (minimap2 PAF-style all-vs-all overlaps with dv tags)
- Identity/sequence-divergence matrix (x = 1 - dv)
- Read clusters from MCL (in Argo's example run: 149 read clusters)

## How to apply

First restrict the graph to reads with ARG hits (filtering ARG-free reads bypasses unnecessary alignments). Overlap the ARG-containing reads with minimap2 in all-vs-all approximate mode ('-x ava-ont'), retaining dv tags to compute per-pair sequence divergence (dv). Compute the median dv over overlaps and discard overlaps with dv > max(2.5 x median dv, 0.05); set the adaptive identity cutoff to 90 - 2.5 x 100 x median dv. Convert x = 1 - dv into an identity matrix and cluster it with MCL using expansion e = 2 and inflation r = 2, pruning entries z < 1/n after each iteration until convergence. Clustering aggregates evidence across locally co-occurring reads, which substantially reduces host misclassification compared with per-read taxonomic classifiers.

## Related tools

- **Argo** (End-to-end profiler that runs the overlap-graph clustering and downstream set-cover species assignment) — https://github.com/xinehc/argo
- **minimap2** (All-vs-all approximate read overlapping ('-x ava-ont') producing overlaps with dv divergence tags)
- **MCL** (Graph clustering (expansion e = 2, inflation r = 2, pruning z < 1/n) that decomposes the overlap graph into read clusters)
- **DIAMOND** (Aligns reads against indexed SARG+ proteins to identify the ARG-containing reads that enter the overlap graph)

## Examples

```
minimap2 -x ava-ont example.fa.gz example.fa.gz > overlaps.paf
```

## Evaluation signals

- Overlaps with dv > max(2.5 x median dv, 0.05) are removed before graph construction
- The adaptive identity cutoff equals 90 - 2.5 x 100 x median dv
- MCL runs with e = 2 and r = 2 and iterates with z < 1/n pruning until convergence
- On the Loman Lab R10.3 mock example (10,000 QC ONT reads), the run should reproduce 149 read clusters
- Downstream output example.sarg.tsv should report 8 ARG-carrying species, 14 ARG types, 138 ARG subtypes, and 24.478 total ARG copies per genome

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- Long-read base-level errors cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded

## Evidence

- [methods] dv-based overlap filtering and adaptive cutoff: [evidence span withheld — anchor status: not_found]
- [methods] MCL parameters: [evidence span withheld — anchor status: not_found]
