---
name: mcl-read-cluster-community-detection
description: Use when profiling ARGs from quality-controlled ONT/PacBio long reads
  and you need species-resolved host assignment by grouping reads via overlap-graph
  community detection before taxonomic labeling.
when_to_use_negative:
- Input is short-read (second-generation) data — Argo's overlap-graph approach is
  designed for long reads where overlaps span ARG flanking regions
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied
  environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2
  + GTDB-Tk) methods may be better suited
- Coverage is insufficient (below ~1x genome copy), where inadequate read overlaps
  make clustering unstable and taxonomy unreliable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Argo
  role: Long-read ARG profiler that orchestrates overlap detection, MCL clustering,
    and per-cluster set-cover taxonomic assignment
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: All-versus-all approximate read overlapping ('-x ava-ont') with dv tags used
    to build the identity matrix
- name: MCL
  role: Markov Clustering algorithm decomposing the read-overlap graph into read clusters
    (expansion e=2, inflation r=2)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:a08143478d4c52c806f787d262244e29bebd2840ab3c0c30c74cc5727616613e
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mcl-read-cluster-community-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster ARG-containing long reads into species-resolved groups by building a read-overlap identity matrix with minimap2 all-versus-all mapping and decomposing the overlap graph with the Markov Clustering (MCL) algorithm, enabling Argo to assign taxonomic labels per cluster rather than per read.

## When to use

Use when profiling ARGs from quality-controlled ONT/PacBio long reads and you need species-resolved host assignment by grouping reads via overlap-graph community detection before taxonomic labeling.

## When NOT to use

- Input is short-read (second-generation) data — Argo's overlap-graph approach is designed for long reads where overlaps span ARG flanking regions
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods may be better suited
- Coverage is insufficient (below ~1x genome copy), where inadequate read overlaps make clustering unstable and taxonomy unreliable

## Inputs

- ARG-containing quality-controlled long reads (Nanopore or PacBio, e.g. FASTA/FASTQ) previously flagged by Argo's ARG detection step
- Argo database with ARG-tailored GTDB/plasmid FASTAs indexed via 'minimap2 -x map-ont -d'
- minimap2 overlap parameters: -x ava-ont preset with dv tagging

## Outputs

- Filtered read-overlap set (dv-filtered minimap2 PAF overlaps)
- Identity matrix (x = 1 - dv) over ARG-containing reads
- Read clusters (MCL communities) that feed per-cluster weighted set-cover taxonomic assignment
- Downstream example.sarg.tsv abundance table of ARG types/subtypes, hosts, and ARG copies per genome (cpg)

## How to apply

First align ARG-containing long reads against each other using minimap2 in approximate-mapping mode ('-x ava-ont'), tagging each overlap with sequence divergence (dv tags). Compute the median dv over all overlaps and discard overlaps where dv > max(2.5 x median dv, 0.05); set the adaptive identity cutoff to 90 - 2.5 x 100 x median dv. Build the identity matrix with entries x = 1 - dv from the filtered overlaps and cluster it with MCL using expansion e = 2 and inflation r = 2, pruning entries z < 1/n after each iteration until convergence. The resulting read clusters consolidate taxonomic evidence: each cluster is later labeled via greedy weighted set cover, so accurate overlap filtering and clustering directly determine host-tracking accuracy. Rationale: because ARGs are prone to horizontal gene transfer across plasmids and chromosomes, per-read classification is unreliable, whereas clustering aggregates evidence from reads sharing the same genomic context.

## Related tools

- **Argo** (Long-read ARG profiler that orchestrates overlap detection, MCL clustering, and per-cluster set-cover taxonomic assignment) — https://github.com/xinehc/argo
- **minimap2** (All-versus-all approximate read overlapping ('-x ava-ont') with dv tags used to build the identity matrix)
- **MCL** (Markov Clustering algorithm decomposing the read-overlap graph into read clusters (expansion e=2, inflation r=2))

## Examples

```
argo example.fa.gz -d database -o . --plasmid
```

## Evaluation signals

- Overlap filtering invariant: no retained overlap has dv > max(2.5 x median dv, 0.05)
- MCL convergence: iterations continue pruning z < 1/n entries until the clustering stabilizes
- On the Loman Lab R10.3 mock example (10,000 QC ONT reads), the run should reproduce 149 read clusters
- Downstream validation: the example produces 8 ARG-carrying species, 14 ARG types, 138 ARG subtypes, and a total of 24.478 ARG copies per genome in example.sarg.tsv
- Species labels per cluster follow the weighted set cover with w(s_j) = -sum(AS_i,j), with HSPs removed when jointly covered < 90% by cluster reads

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- Species with less than one genome copy (1x) are marked 'unclassified' by default, making the approach less suitable for extremely rare species

## Evidence

- [other] MCL parameters and dv filtering: [evidence span withheld — anchor status: not_found]
- [other] MCL clustering step: [evidence span withheld — anchor status: not_found]
