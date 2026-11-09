---
name: containment-collapsing-of-overlapping-read-nodes
description: Use when building HiFiCCL's bidirectional ALG from minimap2 all-vs-all
  HiFi read overlaps and the PAF reveals reads fully aligned inside longer reads,
  to avoid redundant nodes before edge weighting and path search.
when_to_use_negative:
- Reads only partially overlap one another (no containment relationship) — partial
  overlaps remain separate ALG nodes connected by weighted edges
- Input is already assembled contigs or a chromosome-binned read set rather than raw
  HiFi reads with all-vs-all overlaps
- Running an off-the-shelf assembler (e.g., hifiasm) directly without HiFiCCL's reference-guided,
  chromosome-by-chromosome binning workflow
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: HiFiCCL
  role: Implements ALG construction and collapses nodes of shorter contained sequences
    into longer ones before edge weighting and path search
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: minimap2
  role: Produces the all-vs-all overlap PAF from which containment relationships between
    reads are detected
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Python library used in HiFiCCL (version 0.21.0) to parse the reference alignments
    supplying per-read chromosome labels
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:ccbb84884c00b8bb95388bedefd6843e44d245a6c8b63189000fcfb18682551c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - HiFiCCL
  - Python
  - minimap2
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# containment-collapsing-of-overlapping-read-nodes

## Summary

During construction of HiFiCCL's Alignment Label Graph (ALG), any HiFi read that is fully contained within a longer read (per minimap2 all-vs-all overlaps) is collapsed: the node of the shorter sequence is merged into the node of the longer sequence so the overlap graph does not double-count contained fragments.

## When to use

Use when building HiFiCCL's bidirectional ALG from minimap2 all-vs-all HiFi read overlaps and the PAF reveals reads fully aligned inside longer reads, to avoid redundant nodes before edge weighting and path search.

## When NOT to use

- Reads only partially overlap one another (no containment relationship) — partial overlaps remain separate ALG nodes connected by weighted edges
- Input is already assembled contigs or a chromosome-binned read set rather than raw HiFi reads with all-vs-all overlaps
- Running an off-the-shelf assembler (e.g., hifiasm) directly without HiFiCCL's reference-guided, chromosome-by-chromosome binning workflow

## Inputs

- HiFi read set (FASTA/FASTQ)
- minimap2 all-vs-all overlap file in PAF format (prefix.all_vs_all.paf)
- minimap2 reference alignments in SAM format providing initial chromosome labels (prefix.map_to_reference.sam)

## Outputs

- ALG node set with contained shorter reads collapsed into their longer representative nodes
- collapsed-read-to-representative mapping used when propagating chromosome classes during path search

## How to apply

First obtain pairwise overlaps by running minimap2 all-vs-all read mapping with the HiFiCCL parameters '-D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20'. For each overlap, test the containment relationship: if one sequence's alignment spans its full length inside the other, treat the shorter read as contained. Collapse the node of the shorter contained sequence into the node of the longer sequence, redirecting its edges, so each containment group is represented by a single node. Only after collapsing, assign directed forward/reverse edges weighted by the overlap degree od = al^2 * (m/(ql*rl)) * w and label each remaining node with its reference-derived chromosome class or 'None'. Rationale: collapsing prevents contained reads from inflating overlap-degree weights and from spawning spurious paths during the Z-drop-inspired bidirectional search and majority-vote class propagation.

## Related tools

- **HiFiCCL** (Implements ALG construction and collapses nodes of shorter contained sequences into longer ones before edge weighting and path search) — https://github.com/zjjbuqi/HiFiCCL
- **minimap2** (Produces the all-vs-all overlap PAF from which containment relationships between reads are detected) — https://github.com/lh3/minimap2
- **pysam** (Python library used in HiFiCCL (version 0.21.0) to parse the reference alignments supplying per-read chromosome labels) — https://github.com/pysam-developers/pysam

## Examples

```
minimap2 -D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20 reads.fq.gz reads.fq.gz > prefix.all_vs_all.paf
```

## Evaluation signals

- Number of ALG nodes is smaller than the number of input HiFi reads, with the reduction equal to the number of fully contained reads detected in the PAF
- No overlap edge or path references a read ID that was collapsed into a longer representative node
- Each containment group contributes exactly one node to any Z-drop-truncated path, so majority-vote class propagation counts it once
- Downstream chr_by_chr_reads output still covers all input reads, with collapsed reads inheriting their representative's final chromosome class
- Overlap-degree weights od = al^2 * (m/(ql*rl)) * w are computed only on edges between surviving (non-collapsed) nodes

## Limitations

- Only fully contained (containment-relationship) sequences are collapsed; partially overlapping reads remain distinct nodes
- Collapse correctness depends on minimap2 all-vs-all sensitivity; missed or chimeric overlaps can misidentify containment
- At ~5x coverage many genomic regions remain missing, so containment collapsing alone cannot recover unrepresented regions

## Evidence

- [methods] if two sequences exhibit a containment relationship, the node representing the shorter sequence is collapsed into the node of the longer sequence: "if two sequences exhibit a containment relationship, the node representing the shorter sequence is collapsed into the node of the longer sequence"
- [methods] Based on the inter‐sequence comparison information, a bidirectional graph is constructed: "Based on the inter‐sequence comparison information, a bidirectional graph is constructed"
- [readme] ./minimap2 -x ava-pb reads.fa reads.fa > overlaps.paf     # PacBio read overlap: [evidence span withheld — anchor status: external_doc]
