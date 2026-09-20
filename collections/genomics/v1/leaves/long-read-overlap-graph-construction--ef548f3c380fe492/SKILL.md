---
name: long-read-overlap-graph-construction
description: Use when pairwise overlaps of long reads must be converted into a labeled,
  weighted bidirectional graph (nodes=read IDs, edges=overlaps) to support chromosome
  class correction or read binning, e.g., low-coverage HiFi reference-guided assembly.
when_to_use_negative:
- Input reads are short Illumina reads — minimap2 overlap construction here is tuned
  for long reads with error rate up to ~15%, and HiFiCCL targets HiFi long reads
- You already have validated chromosome-binned read sets and only need assembly (e.g.,
  hifiasm can be run directly per bin)
- The goal is graph-based pangenome construction — HiFiCCL also builds a separate
  Pangenome Label Graph (PLG) for NA/HG class standardization, which is a different
  graph type
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Produces the pairwise all-vs-all read overlaps (PAF) and reference chromosome
    labels ('-ax map-hifi') from which the ALG edges and node labels are built
  repo: https://github.com/lh3/minimap2
- name: HiFiCCL
  role: Builds the bidirectional ALG from the minimap2 overlaps, applies overlap-degree
    weighting, containment collapsing, and node class labeling
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: pysam
  role: Python library (0.21.0) used to parse the minimap2 SAM alignments for reference-derived
    chromosome class labeling
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:f307f49514fa3dc8c72d0a2d2a034c8771765e00d463153c14b14e34753b7ca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HiFiCCL
  - Python
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# long-read-overlap-graph-construction

## Summary

Constructs the bidirectional Alignment Label Graph (ALG) used by HiFiCCL: HiFi reads become nodes and minimap2 all-vs-all pairwise overlaps become directed edges weighted by overlap degree od = al^2 * (m/(ql*rl)) * w. This graph is the substrate for de novo chromosome class correction and chromosome-wise read binning at low (~5x) coverage.

## When to use

Use when pairwise overlaps of long reads must be converted into a labeled, weighted bidirectional graph (nodes=read IDs, edges=overlaps) to support chromosome class correction or read binning, e.g., low-coverage HiFi reference-guided assembly.

## When NOT to use

- Input reads are short Illumina reads — minimap2 overlap construction here is tuned for long reads with error rate up to ~15%, and HiFiCCL targets HiFi long reads
- You already have validated chromosome-binned read sets and only need assembly (e.g., hifiasm can be run directly per bin)
- The goal is graph-based pangenome construction — HiFiCCL also builds a separate Pangenome Label Graph (PLG) for NA/HG class standardization, which is a different graph type

## Inputs

- HiFi/CCS long reads (FASTQ/FASTA)
- Pairwise read overlaps from minimap2 all-vs-all mapping (PAF, e.g., *prefix*.all_vs_all.paf)
- Reference-derived chromosome labels from minimap2 '-ax map-hifi' alignments to a linear reference (e.g., CHM13 v2.0), retained as SAM (*prefix*.map_to_reference.sam)

## Outputs

- Bidirectional Alignment Label Graph (ALG) with read-ID nodes, directed forward/reverse edges, and weights od = al^2 * (m/(ql*rl)) * w
- Node labels: reference chromosome class ('chr*' strings) or 'None' for unaligned reads
- Collapsed graph where shorter contained reads are merged into longer read nodes

## How to apply

First run minimap2 all-vs-all read mapping with '-D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20' to obtain pairwise overlaps (PAF). Build the bidirectional ALG with nodes = read IDs and directed forward/reverse edges weighted by the overlap degree od = al^2 * (m/(ql*rl)) * w, collapsing nodes of shorter contained sequences into the nodes of longer ones; label each node with its reference-derived chromosome class (from minimap2 '-ax map-hifi' alignments, e.g., against CHM13 v2.0) or 'None' if the read did not align. Rationale: overlap degree weighting lets the downstream Z-drop-inspired heuristic partition the graph into reliable linear subgraphs, and containment collapsing removes redundant short reads while preserving their overlap information.

## Related tools

- **minimap2** (Produces the pairwise all-vs-all read overlaps (PAF) and reference chromosome labels ('-ax map-hifi') from which the ALG edges and node labels are built) — https://github.com/lh3/minimap2
- **HiFiCCL** (Builds the bidirectional ALG from the minimap2 overlaps, applies overlap-degree weighting, containment collapsing, and node class labeling) — https://github.com/zjjbuqi/HiFiCCL
- **pysam** (Python library (0.21.0) used to parse the minimap2 SAM alignments for reference-derived chromosome class labeling) — https://github.com/pysam-developers/pysam

## Examples

```
minimap2 -D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20 hifi_reads.fq hifi_reads.fq > prefix.all_vs_all.paf
```

## Evaluation signals

- The PAF of pairwise overlaps (*prefix*.all_vs_all.paf) exists and every graph edge can be traced to a PAF overlap record
- Edge weights match the overlap-degree formula od = al^2 * (m/(ql*rl)) * w for the underlying overlap lengths
- Containment relationships are collapsed: no graph node corresponds to a sequence strictly contained in a longer read
- Every node carries either a reference chromosome label ('chr*' string) or 'None' consistent with its minimap2 map-hifi SAM alignment
- Both forward and reverse directed edges exist for each overlap (bidirectional graph), with --dual=no suppressing duplicate dual alignments in the underlying mapping

## Limitations

- No version numbers are stated for HiFiCCL in the source, so graph-construction behavior may drift across repository revisions (no changelog found)
- At low (~5x) coverage the overlap graph is sparse, which motivates the overlap-degree weighting but limits how reliable the resulting linear subgraphs can be
- Overlap detection via minimap2 is bounded by its ~15% error-rate range, so very divergent sequence is not represented as edges

## Evidence

- [methods] Based on the inter‐sequence comparison information, a bidirectional graph is constructed: [evidence span withheld — anchor status: not_found]
- [methods] if two sequences exhibit a containment relationship, the node representing the shorter sequence is collapsed into the node of the longer sequence: [evidence span withheld — anchor status: not_found]
- [methods] Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes: "Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes"
- [readme] (2) finding overlaps between long reads with error rate up to ~15%: [evidence span withheld — anchor status: external_doc]
