---
name: frequent-itemset-graph-pruning
description: Use when building/pruning an ad-hoc variation graph from a long-read
  alignment pile and you must remove error-induced nodes/edges while preserving true
  low-frequency haplotype variants.
when_to_use_negative:
- Deriving haplotype-aware consensus sequences in cycle 2 step 4 — VeChat derives
  these by dynamic programming without graph re-pruning, so this pruning skill does
  not apply there
- Input reads are already high-accuracy (e.g. PacBio HiFi/CCS) — VeChat targets noisy
  TGS long reads where error-induced graph elements dominate
- Computational resources are the binding constraint on very large datasets — VeChat
  experiences longer runtimes and higher peak memory usage than competing correctors
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: VeChat
  role: Implements the iterative frequent-itemset pruning of the variation graph;
    -d and -s flags set the confidence/support thresholds
  repo: https://github.com/HaploKit/vechat
- name: Racon
  role: Its modified POA module constructs the ad-hoc variation graph that is subsequently
    pruned
  repo: https://github.com/lbcb-sci/racon
- name: Minimap2
  role: Computes the minimizer-based all-vs-all overlaps that define the read alignment
    piles feeding graph construction
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:2053c26ceacd52d090a9af62de360c9f9315eac2e3a03a23d887466b1d48b034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - Racon
  - VeChat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# frequent-itemset-graph-pruning

## Summary

VeChat's step-4 procedure that prunes sequencing-error artifacts from an ad-hoc variation graph (built via POA from a read alignment pile) using a frequent itemset model over read coverage, sequencing errors and character co-occurrence, with directional Confidence and Support thresholds per edge.

## When to use

Use when building/pruning an ad-hoc variation graph from a long-read alignment pile and you must remove error-induced nodes/edges while preserving true low-frequency haplotype variants.

## When NOT to use

- Deriving haplotype-aware consensus sequences in cycle 2 step 4 — VeChat derives these by dynamic programming without graph re-pruning, so this pruning skill does not apply there
- Input reads are already high-accuracy (e.g. PacBio HiFi/CCS) — VeChat targets noisy TGS long reads where error-induced graph elements dominate
- Computational resources are the binding constraint on very large datasets — VeChat experiences longer runtimes and higher peak memory usage than competing correctors

## Inputs

- Raw long-read input in FASTA/FASTQ format (gzip-compressed accepted, e.g. example/reads.fq.gz) from PacBio CLR or ONT sequencing
- Per-window read alignment pile computed for the selected target read
- Ad-hoc variation graph built via POA, with edges weighted by read coverage and per-base Phred probabilities

## Outputs

- Pruned variation graph containing only edges passing Confidence(e) >= 0.2 and Support(e) >= 0.2
- Corrected target subread (optimal alignment path through the converged graph)
- Convergence status of the iterative prune-and-realign cycle (up to 3 iterations)

## How to apply

Construct the variation graph for each window of the read alignment pile using the partial order alignment (POA) routine from the Racon module bundled with VeChat, then annotate nodes and edges with weights from read coverage and per-base Phred probabilities under a frequent itemset model involving read coverage, sequencing errors and co-occurrence of characters in reads. For every edge compute directional Confidence(e) and Support(e); prune any edge below the minimum-confidence threshold (-d, default 0.2) or the minimum-support threshold (-s, default 0.2). Re-align the target subread against the pruned graph and repeat the prune-and-realign cycle until convergence (3 iterations), so the surviving graph contains only nodes and edges reflecting true sequential phenomena. Read off the optimal alignment path of the target subread through the converged graph as the corrected subread.

## Related tools

- **VeChat** (Implements the iterative frequent-itemset pruning of the variation graph; -d and -s flags set the confidence/support thresholds) — https://github.com/HaploKit/vechat
- **Racon** (Its modified POA module constructs the ad-hoc variation graph that is subsequently pruned) — https://github.com/lbcb-sci/racon
- **Minimap2** (Computes the minimizer-based all-vs-all overlaps that define the read alignment piles feeding graph construction) — https://github.com/lh3/minimap2

## Examples

```
vechat -u example/reads.fq.gz -d 0.2 -s 0.2
```

## Evaluation signals

- Every surviving edge satisfies Confidence(e) >= 0.2 (-d default) and Support(e) >= 0.2 (-s default)
- The prune-and-realign cycle converges within 3 iterations; non-convergence flags incorrect thresholds or graph construction
- Upon termination the graph exclusively contains nodes/edges reflecting true sequential phenomena rather than sequencing-error artifacts
- The corrected subread is exactly the optimal alignment path of the target subread through the converged graph
- Downstream benchmark error rates should drop relative to raw reads (VeChat reports 4–15x fewer errors on PacBio, 1–10x on ONT vs state-of-the-art correctors)

## Limitations

- VeChat does not win on computational resources: on large datasets it shows longer runtimes and higher peak memory usage
- It uses off-the-shelf approaches (e.g. edit-distance and sequence-to-graph alignments) without exploiting all their features, adding avoidable overheads
- Optimal pipeline combinations (preprocessing/correction/assembly) for polyploid, cancer or metagenomic settings remain to be explored

## Evidence

- [results] Frequent itemset model definition: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [results] Iterative convergence: "The procedure of spurious node and edge removal followed by re-alignment is repeated until convergence"
- [results] POA graph construction and pruning: "pruning this graph in an iterative manner from nodes and edges that are spurious"
- [methods] Confidence threshold CLI parameter: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [methods] Support threshold CLI parameter: "-s MIN_SUPPORT, --min-support MIN_SUPPORT minimum support for keeping edges in the graph (default: 0.2)"
- [readme] README description of pruning rationale: "nodes and edges that are due to errors are pruned from that graph by way of an iterative procedure that is based on principles from frequent itemset mining"
- [methods] Example test input: "One can test the program using the small sequencing read file `example/reads.fq.gz`"
