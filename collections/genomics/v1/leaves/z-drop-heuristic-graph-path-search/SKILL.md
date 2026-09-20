---
name: z-drop-heuristic-graph-path-search
description: Use when low-coverage (~5x) HiFi reads are misclassified or unclassified
  ('None') after minimap2 alignment to a linear reference, and overlap-graph-based
  de novo chromosome-class correction is needed before per-chromosome assembly.
when_to_use_negative:
- Reads already carry reliable high-coverage per-chromosome labels (e.g., deep-coverage
  data where alignment-based binning alone suffices) — the correction targets misclassified/None
  reads arising at ~5x coverage
- Correcting inter-chromosomal translocation breakpoints — that is handled by the
  separate translocation correction module (SigBND extraction and BND signal clustering),
  not the de novo ALG path search
- Noisy non-HiFi reads (ONT/CLR) — the overlap-degree weighting and containment collapsing
  assume high-fidelity reads; noisy-read assemblers (Canu, Flye, wtdbg) are a different
  pipeline
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiFiCCL
  role: Implements the Z-drop-inspired ALG search, per-path majority-vote propagation,
    and multi-iteration class voting (-W 0.75, -N 3, --iterations 200)
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: minimap2
  role: Produces the all-vs-all read overlaps (PAF) and reference alignments (SAM)
    used to construct and label the ALG edges and nodes
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Python library used by HiFiCCL to read minimap2 SAM alignment labels when
    building and labeling the graph
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:26e785b5a91388672aed30ae8758dcc8f79ec9077fc19d97cd411346a91573f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - HiFiCCL
  - Python
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# z-drop-heuristic-graph-path-search

## Summary

HiFiCCL's de novo class correction: a Z-drop-inspired heuristic searches the bidirectional Alignment Label Graph (ALG) of HiFi reads, truncating greedy paths when edge weights drop significantly, then reassigns chromosome classes by majority vote per path and across randomized iterations to fix misclassified or 'None'-labeled reads under low (~5x) coverage.

## When to use

Use when low-coverage (~5x) HiFi reads are misclassified or unclassified ('None') after minimap2 alignment to a linear reference, and overlap-graph-based de novo chromosome-class correction is needed before per-chromosome assembly.

## When NOT to use

- Reads already carry reliable high-coverage per-chromosome labels (e.g., deep-coverage data where alignment-based binning alone suffices) — the correction targets misclassified/None reads arising at ~5x coverage
- Correcting inter-chromosomal translocation breakpoints — that is handled by the separate translocation correction module (SigBND extraction and BND signal clustering), not the de novo ALG path search
- Noisy non-HiFi reads (ONT/CLR) — the overlap-degree weighting and containment collapsing assume high-fidelity reads; noisy-read assemblers (Canu, Flye, wtdbg) are a different pipeline

## Inputs

- PacBio HiFi FASTQ/FASTA reads (e.g., 5x coverage human datasets such as HG002 SRR10382244)
- minimap2 all-vs-all pairwise overlaps in PAF format (*prefix*.all_vs_all.paf)
- minimap2 reference alignments in SAM format (*prefix*.map_to_reference.sam) providing initial chromosome labels
- Linear reference genome (e.g., CHM13 v2.0) for initial class assignment

## Outputs

- Corrected final chromosome class per read (majority vote over ClassHash across M iterations)
- Per-chromosome read bins written to chr_by_chr_reads, ready for per-chromosome hifiasm assembly
- Intermediate ALG/graph structures with overlap-degree-weighted edges and ClassHash per-node class lists

## How to apply

Build the bidirectional ALG from minimap2 all-vs-all read overlaps ('-D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20'), weighting directed edges by the overlap degree od = al^2 * (m/(ql*rl)) * w and collapsing contained (shorter) sequence nodes into longer ones; label nodes with reference-derived chromosome classes or 'None'. Then run M randomized-order iterations (default --iterations 200) of the Z-drop-inspired search: from each unvisited node, greedily extend a bidirectional path over unvisited neighbors prioritized by edge weight, truncating when Equation (2) w_jq <= theta * w_ij + epsilon is violated (default Z-drop weight -W 0.75, minimum supporting reads -N 3). For each resulting linear subgraph (path), update chromosome classes by majority vote (Equation 3) and propagate the winning class to all path nodes only when the path exceeds the minimum support threshold N(PATH_k) > S_N_min, recording per-node class lists in ClassHash across iterations. Finally assign each read's class by majority vote across the M iterations (Equation 5) and cluster reads into chromosomal bins (Equation 6). The rationale is that reliable same-chromosome read chains show stable high overlap weights, while a sharp weight drop signals a spurious or inter-chromosomal overlap, so truncation isolates trustworthy linear subgraphs even at 5x coverage.

## Related tools

- **HiFiCCL** (Implements the Z-drop-inspired ALG search, per-path majority-vote propagation, and multi-iteration class voting (-W 0.75, -N 3, --iterations 200)) — https://github.com/zjjbuqi/HiFiCCL
- **minimap2** (Produces the all-vs-all read overlaps (PAF) and reference alignments (SAM) used to construct and label the ALG edges and nodes) — https://github.com/lh3/minimap2
- **pysam** (Python library used by HiFiCCL to read minimap2 SAM alignment labels when building and labeling the graph) — https://github.com/pysam-developers/pysam

## Examples

```
minimap2 -D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20 reads.fq reads.fq > prefix.all_vs_all.paf
```

## Evaluation signals

- chr_by_chr_reads output exists and contains per-chromosome read bins covering all input reads
- Every read within each bin carries the same final chromosome label consistent with the majority-vote result (Equation 5)
- Class propagation to a path's nodes occurred only for paths exceeding the minimum support threshold N(PATH_k) > S_N_min (default -N 3)
- Path extension stopped exactly where Equation (2) w_jq <= theta * w_ij + epsilon was first violated (theta = -W 0.75), i.e., truncated paths end at sharp edge-weight drops
- Per-node ClassHash entries across the M = 200 randomized iterations converge to a stable winning class per read

## Limitations

- At 5x coverage many genomic regions remain missing, so correction cannot recover classes for reads whose chromosome has too few overlapping neighbors
- Random node selection/order makes results stochastic; convergence relies on M randomized iterations (default 200) and majority voting rather than a deterministic guarantee
- Reference-based initial labels can be wrong for true SVs, and the de novo module corrects classes but reference-guided scaffolding may still overlook true SVs (e.g., only 62.5% of inversion SVs detected by RagTag scaffolding)

## Evidence

- [methods] Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes: "Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes"
- [methods] Majority vote per path: "the chromosomal class is updated based on the majority vote among the nodes in the path"
- [methods] Containment collapsing: "if two sequences exhibit a containment relationship, the node representing the shorter sequence is collapsed into the node of the longer sequence"
- [other] Truncation on edge-weight drop: [evidence span withheld — anchor status: not_found]
- [methods] Binning of corrected reads: "reads are clustered into chromosomal bins based on their final assigned classes"
