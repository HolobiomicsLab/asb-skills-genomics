---
name: sequence-to-graph-realignment-iteration
description: Use when a per-window read alignment pile has been turned into a POA
  variation graph and sequencing-error-induced nodes/edges must be pruned iteratively
  before reading off the corrected subread's alignment path.
when_to_use_negative:
- 'Cycle-2 haplotype-aware consensus derivation: consensus sequences are derived from
  the constructed variation graphs by dynamic programming without re-pruning the graph'
- Very large datasets where VeChat's longer runtimes and higher peak memory usage
  make it computationally unattractive versus consensus-based correctors
- Input is not raw noisy long reads (VeChat is a self-correction method for raw TGS
  reads, not a polishing step for finished assemblies)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Implements the iterative confidence/support graph pruning and sequence-to-graph
    re-alignment loop
  repo: https://github.com/HaploKit/vechat
- name: Racon
  role: Its modified POA routine constructs the ad-hoc variation graph that is then
    iteratively pruned
  repo: https://github.com/lbcb-sci/racon
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:a98a90dc40e2993cbb64398fdd4b81b6eb0bb1c8afb70b787085ce73d3fd8ebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Racon
  - VeChat
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sequence-to-graph-realignment-iteration

## Summary

Iterative prune-and-realign cycle for ad-hoc variation graphs built by POA: edges lacking sufficient directional Confidence(e) and Support(e) from a frequent itemset model are removed, the target subread is re-aligned, and the cycle repeats until the graph retains only true sequential phenomena.

## When to use

Use when a per-window read alignment pile has been turned into a POA variation graph and sequencing-error-induced nodes/edges must be pruned iteratively before reading off the corrected subread's alignment path.

## When NOT to use

- Cycle-2 haplotype-aware consensus derivation: consensus sequences are derived from the constructed variation graphs by dynamic programming without re-pruning the graph
- Very large datasets where VeChat's longer runtimes and higher peak memory usage make it computationally unattractive versus consensus-based correctors
- Input is not raw noisy long reads (VeChat is a self-correction method for raw TGS reads, not a polishing step for finished assemblies)

## Inputs

- Raw long-read file in FASTA/FASTQ format (gzip-compressed accepted, e.g. example/reads.fq.gz)
- Per-window read alignment pile for the selected target read
- Ad-hoc variation graph constructed via POA from the modified Racon module

## Outputs

- Pruned variation graph containing only edges passing the confidence (0.2) and support (0.2) thresholds
- Corrected target subread, i.e. the optimal alignment path of the target subread through the converged graph

## How to apply

For each window of the read alignment pile, construct an ad-hoc variation graph with the partial order alignment (POA) routine from the modified Racon module bundled with VeChat. Annotate nodes and edges with weights from read coverage and per-base Phred probabilities, applying a frequent itemset model over read coverage, sequencing errors, and co-occurrence of characters in reads to flag error-induced elements. Compute the directional Confidence(e) and Support(e) of each edge and prune any edge below the minimum-confidence threshold (-d, default 0.2) or minimum-support threshold (-s, default 0.2). Re-align the target subread against the pruned graph and repeat the prune-and-realign cycle until convergence (3 iterations), so that upon termination the graph exclusively contains nodes and edges reflecting true sequential phenomena. Read off the optimal alignment path of the target subread through the converged graph as the corrected subread.

## Related tools

- **VeChat** (Implements the iterative confidence/support graph pruning and sequence-to-graph re-alignment loop) — https://github.com/HaploKit/vechat
- **Racon** (Its modified POA routine constructs the ad-hoc variation graph that is then iteratively pruned) — https://github.com/lbcb-sci/racon

## Examples

```
vechat -i example/reads.fq.gz -d 0.2 -s 0.2
```

## Evaluation signals

- Pruning converges within the default 3 iterations under -d 0.2 and -s 0.2 thresholds
- Surviving graph contains only edges whose directional Confidence(e) >= 0.2 and Support(e) >= 0.2
- The corrected subread corresponds to the optimal alignment path through the converged graph
- Iterated node/edge removal followed by re-alignment terminates rather than oscillating
- Error-induced elements flagged by the frequent itemset model over coverage, errors and character co-occurrence are removed

## Limitations

- VeChat does not win the competition in terms of computational resources: on large datasets it experiences longer runtimes and higher peak memory usage
- Off-the-shelf routines (edit-distance-based and sequence-to-graph alignments) are used without all their features, causing unnecessary overheads
- Optimal pipeline combinations with preprocessing/assembly for polyploid, cancer, or metagenomic settings are beyond the study's scope

## Evidence

- [results] construction of a variation graph using POA and iterative pruning: "the construction of a variation graph using the partial order alignment (POA) algorithm30, and pruning this graph in an iterative manner from nodes and edges that are spurious"
- [results] pruning followed by re-alignment repeats until convergence: "The procedure of spurious node and edge removal followed by re-alignment is repeated until convergence"
- [results] frequent itemset model over coverage, errors, co-occurrence: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [methods] default confidence and support CLI thresholds for edge retention: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [readme] README: iterative error-node/edge pruning based on frequent itemset mining: "nodes and edges that are due to errors are pruned from that graph by way of an iterative procedure that is based on principles from frequent itemset mining."
