---
name: bubble-region-identification
description: Use when you have minigraph-constructed pangenome graphs in GFA format
  and need to extract bubble regions as BED intervals to compare graph structure against
  a reference pangenome (e.g., HPRC).
when_to_use_negative:
- Input is raw HiFi reads rather than an assembled, graph-ready set of contigs — assemble
  first (e.g., HiFiCCL-Hifiasm) before graph construction
- The goal is per-sample structural variant genotyping/calling rather than graph-level
  bubble comparison — use a dedicated SV caller (e.g., minigraph path calling or minisv)
  instead
- No reference pangenome graph (e.g., HPRC) or its bubble BED exists to serve as ground
  truth for TP/FP/FN classification
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: gfatools
  role: Parses the minigraph-produced GFA graph and emits bubble regions as a BED
    file (gfa2bed) for downstream overlap-based benchmarking
  repo: https://github.com/lh3/gfatools
- name: minigraph
  role: Constructs the pangenome graph incrementally from GRCh38 as reference plus
    CHM13 and the 45 per-sample assemblies; also builds the HPRC reference graph used
    as ground truth
  repo: https://github.com/lh3/minigraph
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:583712164a59db1dcb7419859c9e26edec5398f7ee428d025bf3296b34971b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/gfatools
  tools:
  - HiFiCCL
  - gfatools
  - minigraph
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bubble-region-identification

## Summary

Identifies bubble (variant) regions in pangenome graphs built with minigraph from low-coverage (~5×) assemblies, converts them to BED via gfatools, and benchmarks them against an HPRC reference graph using precision/recall/F1.

## When to use

Use when you have minigraph-constructed pangenome graphs in GFA format and need to extract bubble regions as BED intervals to compare graph structure against a reference pangenome (e.g., HPRC).

## When NOT to use

- Input is raw HiFi reads rather than an assembled, graph-ready set of contigs — assemble first (e.g., HiFiCCL-Hifiasm) before graph construction
- The goal is per-sample structural variant genotyping/calling rather than graph-level bubble comparison — use a dedicated SV caller (e.g., minigraph path calling or minisv) instead
- No reference pangenome graph (e.g., HPRC) or its bubble BED exists to serve as ground truth for TP/FP/FN classification

## Inputs

- Pangenome graph in GFA/rGFA format produced by minigraph (incremental -cxggs construction)
- GRCh38 reference sequence and CHM13 (v1.0) sequence
- Per-sample assemblies (~5× HiFi, from HiFiCCL-Hifiasm or Hifiasm) with uniquely renamed sequences, sorted alphabetically
- Reference bubble BED file generated from the HPRC pangenome graph

## Outputs

- BED file of bubble regions per constructed pangenome graph (via gfatools gfa2bed)
- bubble_consistency_metrics.csv with interval-wise Precision, Recall, and F1 values

## How to apply

Build the pangenome graph with minigraph using the GRCh38 sequence as the reference and supplying the CHM13 (v1.0) sequence plus the 45 per-sample (~5×) assemblies — renamed to ensure cross-sample uniqueness and sorted alphabetically — as input. Run gfatools (gfa2bed) on each constructed pangenome graph to identify bubble regions and emit them as a BED file. Compare each BED file against the BED derived from the minigraph graph built from the HPRC release, classifying bubbles with over 80% overlap with the reference as true positives, non-overlapping predicted bubbles as false positives, and reference bubbles lacking 80% overlap as false negatives. Compute Precision = TP/(TP+FP), Recall = TP/(TP+FN), and F1 at multiple overlap-coverage thresholds; in the source study, HiFiCCL-Hifiasm_graph reached 0.9067 precision / 0.6974 recall at the >0% threshold and 0.8435 / 0.5399 at >50%, consistently outperforming Hifiasm_graph.

## Related tools

- **gfatools** (Parses the minigraph-produced GFA graph and emits bubble regions as a BED file (gfa2bed) for downstream overlap-based benchmarking) — https://github.com/lh3/gfatools
- **minigraph** (Constructs the pangenome graph incrementally from GRCh38 as reference plus CHM13 and the 45 per-sample assemblies; also builds the HPRC reference graph used as ground truth) — https://github.com/lh3/minigraph

## Examples

```
./gfatools gfa2bed -m pangenome.gfa > bubbles.bed
```

## Evaluation signals

- All predicted bubbles classified correctly: TP requires >80% overlap with a reference bubble, FP are non-overlapping predicted bubbles, FN are reference bubbles lacking 80% overlap
- Both graphs under comparison yield positive precision and recall at all thresholds (reported >0%-threshold values ~0.9067 precision / ~0.6974 recall; >50%-threshold ~0.8435 / ~0.5399)
- HiFiCCL-Hifiasm_graph metrics must outperform Hifiasm_graph consistently across all bubble-coverage thresholds
- Interval-wise Precision = TP/(TP+FP), Recall = TP/(TP+FN), F1 = 2×Precision×Recall/(Precision+Recall) are all written to bubble_consistency_metrics.csv
- gfatools successfully parses the GFA and the emitted BED intervals correspond to graph bubble regions

## Limitations

- At 5× coverage many genomic regions remain missing, so assemblies are incomplete and some reference bubbles are inevitably missed (recall ~0.70 even for HiFiCCL-Hifiasm)
- The 80%-overlap TP definition is stringent; results are threshold-dependent and must be reported across multiple overlap-coverage intervals
- Correctness depends on renaming each sequence to ensure uniqueness across all 45 assemblies and supplying inputs sorted alphabetically to minigraph

## Evidence

- [readme] gfatools is a set of tools for manipulating sequence graphs in the GFA or the [rGFA][rGFA] format: "gfatools is a set of tools for manipulating sequence graphs in the GFA or the
[rGFA][rGFA] format."
- [readme] ./gfatools gfa2bed -m test/MT.gfa > MT.bed: [evidence span withheld — anchor status: external_doc]
- [methods] the input data included the CHM13 (v1.0) sequence: "the input data included the CHM13 (v1.0) sequence"
- [methods] using the GRCh38 sequence as the reference: "using the GRCh38 sequence as the reference"
- [intro] We validate the effectiveness of HiFiCCL across various applications, including contig-level assembly, large germline SVs detection: "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp), chromosome‐level scaffolding, construction"
- [discussion] due to low coverage and other limitations, many genomic regions remain missing: "due to low coverage and other limitations, many genomic regions remain missing"
