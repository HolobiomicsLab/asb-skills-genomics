---
name: greedy-weighted-set-cover-taxonomic-labeling
description: Use when a long-read metagenomic pipeline must assign species labels
  to groups of ARG-carrying reads (e.g., MCL read clusters from an overlap graph)
  instead of classifying each read individually with a classifier like Kraken2 or
  Centrifuge.
when_to_use_negative:
- The goal is de novo identification of ARG genomic contexts or hosts in new, understudied
  environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 +
  GTDB-Tk) methods may be better suited
- Species present are unseen in the reference database or have novel plasmid syntenies
  — Argo is inherently reference-based and may misclassify them
- 'Coverage is extremely low (below ~1x genome copy): species ARGs default to ''unclassified'',
  and insufficient coverage yields inadequate read overlaps and unstable clustering/normalization'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0798
tools:
- name: Argo
  role: Implements overlap-graph clustering and greedy weighted set cover to assign
    species labels to read clusters and produce the cpg abundance table
  repo: github.com/xinehc/argo
- name: minimap2
  role: Approximate read overlapping ('-x ava-ont') to build the overlap graph and
    base-level alignment to GTDB/plasmid references for raw species assignments
- name: DIAMOND
  role: Protein alignment of reads against indexed SARG+ proteins, providing the AS
    scores used as set-cover weights
- name: MCL
  role: Graph clustering (expansion e = 2, inflation r = 2) of the identity matrix
    to decompose the read-overlap graph into clusters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:cfdafe28c912591d075c7d1bfa9fba59587a9708bcbd2c9ea8afa0de6f4c2838
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - MCL
  - Melon
  - minimap2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# greedy-weighted-set-cover-taxonomic-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns taxonomic labels (species) to clusters of ARG-containing long reads collectively, rather than per-read, by greedily solving a weighted set cover over species sets whose weights are negative sums of DIAMOND alignment scores (AS). This is the cluster-level species-labeling core of Argo.

## When to use

Use when a long-read metagenomic pipeline must assign species labels to groups of ARG-carrying reads (e.g., MCL read clusters from an overlap graph) instead of classifying each read individually with a classifier like Kraken2 or Centrifuge.

## When NOT to use

- The goal is de novo identification of ARG genomic contexts or hosts in new, understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods may be better suited
- Species present are unseen in the reference database or have novel plasmid syntenies — Argo is inherently reference-based and may misclassify them
- Coverage is extremely low (below ~1x genome copy): species ARGs default to 'unclassified', and insufficient coverage yields inadequate read overlaps and unstable clustering/normalization

## Inputs

- Quality-controlled long reads (Nanopore ONT or PacBio) in FASTA/FASTQ
- Read-overlap graph / MCL read clusters of ARG-containing reads (identity matrix from minimap2 'ava-ont' overlaps with dv filtering)
- Argo database: SARG+ protein sequences indexed with 'diamond makedb' and ARG-tailored GTDB/plasmid FASTAs indexed with 'minimap2 -x map-ont -d' (database.tar.gz from Zenodo record 20451539)
- Per-read alignment scores (AS) of cluster reads against species reference sequences

## Outputs

- Species-level taxonomic labels per read cluster
- Filtered, non-redundant HSP set per cluster (HSPs with <90% joint reference coverage removed)
- Per-cluster species labels feeding the ARG copies-per-genome (cpg) abundance table (e.g., example.sarg.tsv)

## How to apply

First cluster ARG-containing reads by decomposing the read-overlap graph: overlap reads with minimap2 approximate mapping ('-x ava-ont'), compute sequence divergence (dv), discard overlaps where dv > max(2.5 x median dv, 0.05), set the adaptive identity cutoff to 90 - 2.5 x 100 x median dv, build the identity matrix (x = 1 - dv), and cluster with MCL (expansion e = 2, inflation r = 2, pruning entries z < 1/n each iteration until convergence). Align cluster reads to SARG+ proteins with DIAMOND and GTDB/plasmid references with minimap2 to collect per-reference alignment scores. For each read cluster, define species sets from prior species assignments and solve the weighted set cover greedily with weight w(s_j) = -sum(AS_i,j), so species that jointly explain the cluster's high-scoring alignments cover the cluster with maximal aggregate alignment evidence. After covering, remove HSPs whose reference sequences are jointly covered by less than 90% by all reads in the cluster and filter redundant HSPs per read. Rationale: aggregating evidence over a local read cluster suppresses per-read misclassifications that plague taxonomic classifiers on short ARG reads, while avoiding assembly.

## Related tools

- **Argo** (Implements overlap-graph clustering and greedy weighted set cover to assign species labels to read clusters and produce the cpg abundance table) — github.com/xinehc/argo
- **minimap2** (Approximate read overlapping ('-x ava-ont') to build the overlap graph and base-level alignment to GTDB/plasmid references for raw species assignments)
- **DIAMOND** (Protein alignment of reads against indexed SARG+ proteins, providing the AS scores used as set-cover weights)
- **MCL** (Graph clustering (expansion e = 2, inflation r = 2) of the identity matrix to decompose the read-overlap graph into clusters)

## Examples

```
argo example.fa.gz -d database -o . --plasmid
```

## Evaluation signals

- On the documented example run (10,000 QC ONT reads from the Loman Lab R10.3 mock), the output example.sarg.tsv reproduces 149 read clusters, 8 ARG-carrying species, 14 ARG types, 138 ARG subtypes, and 24.478 total ARG copies per genome
- Every read cluster in the output receives a species label (or is explicitly 'unclassified' for species below 1x genome copy) rather than per-read labels
- HSPs whose reference sequences are jointly covered by less than 90% by all reads in the cluster are absent from the final HSP set
- No redundant HSPs remain per read after the redundancy filter
- Overlaps used for clustering satisfy dv <= max(2.5 x median dv, 0.05), and cluster identity cutoffs equal 90 - 2.5 x 100 x median dv

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the database
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing
- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- By default, Argo considers ARGs of species with less than one genome copy (1x) as 'unclassified', making it less suitable for extremely rare species

## Evidence

- [full_text] set cover weight definition and cluster workflow: [evidence span withheld — anchor status: not_found]
- [full_text] MCL clustering parameters and dv filtering: [evidence span withheld — anchor status: not_found]
- [full_text] HSP coverage filter threshold: [evidence span withheld — anchor status: not_found]
