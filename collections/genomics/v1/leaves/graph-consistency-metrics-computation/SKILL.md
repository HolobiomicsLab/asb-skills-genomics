---
name: graph-consistency-metrics-computation
description: Use when a pangenome graph built with minigraph (e.g. from low-coverage
  HiFi assemblies) must be quantitatively compared against a reference graph such
  as the HPRC graph, via bubble BED overlap and precision/recall/F1.
when_to_use_negative:
- Input is a single-sample assembly or a contig-level QUAST-style comparison rather
  than a multi-sample pangenome graph in GFA/rGFA format.
- The goal is calling or genotyping structural variants from an existing graph (e.g.
  with minisv) rather than measuring graph-construction consistency against a reference
  graph.
- No reference pangenome graph (e.g. HPRC release) exists for the species/panel, so
  TP/FP/FN bubble classification against a reference BED is impossible.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: minigraph
  role: Constructs the pangenome graph from GRCh38 reference plus CHM13 (v1.0) and
    the 45 per-sample assemblies; also built the HPRC reference graph used for comparison
  repo: https://github.com/lh3/minigraph
- name: gfatools
  role: Parses the constructed pangenome graph (GFA/rGFA) and emits bubble regions
    as a BED file for overlap-based TP/FP/FN classification
  repo: https://github.com/lh3/gfatools
- name: HiFiCCL
  role: Produces the HiFiCCL-Hifiasm low-coverage (~5x) assemblies whose graph is
    the treatment arm being evaluated
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Produces the baseline Hifiasm low-coverage assemblies whose graph serves as
    the comparison arm
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:59c092da09c92095c022e2f0701e1b8ba34f07b3fc6175ee56ff2855ccbebce3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minigraph
  tools:
  - HiFiCCL
  - gfatools
  - hifiasm
  - minigraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# graph-consistency-metrics-computation

## Summary

Quantifies how faithfully a minigraph-constructed pangenome graph (from HiFiCCL-Hifiasm or Hifiasm assemblies of 45 ~5x human genomes) reproduces the HPRC reference pangenome graph by measuring bubble-level precision, recall, and F1.

## When to use

Use when a pangenome graph built with minigraph (e.g. from low-coverage HiFi assemblies) must be quantitatively compared against a reference graph such as the HPRC graph, via bubble BED overlap and precision/recall/F1.

## When NOT to use

- Input is a single-sample assembly or a contig-level QUAST-style comparison rather than a multi-sample pangenome graph in GFA/rGFA format.
- The goal is calling or genotyping structural variants from an existing graph (e.g. with minisv) rather than measuring graph-construction consistency against a reference graph.
- No reference pangenome graph (e.g. HPRC release) exists for the species/panel, so TP/FP/FN bubble classification against a reference BED is impossible.

## Inputs

- Pangenome graphs in GFA/rGFA format constructed by minigraph (HiFiCCL-Hifiasm_graph, Hifiasm_graph, HPRC reference graph)
- Input FASTA set: GRCh38 reference sequence, CHM13 (v1.0) sequence, and 45 uniquely renamed per-sample assemblies (~5x coverage) sorted alphabetically
- HPRC-release pangenome graph BED file of bubble regions

## Outputs

- Per-graph bubble BED files emitted by gfatools
- bubble_consistency_metrics.csv with interval-wise (e.g. >0% and >50% overlap) Precision, Recall, and F1 per assembler variant

## How to apply

Build each pangenome graph with minigraph using the GRCh38 sequence as the reference and supplying the CHM13 (v1.0) sequence plus the 45 per-sample assemblies (HiFiCCL-Hifiasm or Hifiasm, uniquely renamed, sorted alphabetically) as input. Run gfatools (gfa2bed) on each constructed graph to identify bubble regions and emit them as a BED file, and do the same for the HPRC reference graph. Compare predicted vs reference BED files, classifying bubbles with over 80% overlap with the reference as true positives, non-overlapping predicted bubbles as false positives, and reference bubbles lacking 80% overlap as false negatives. Compute Precision = TP/(TP+FP), Recall = TP/(TP+FN), and F1 = 2*Precision*Recall/(Precision+Recall) at multiple bubble coverage thresholds (>0% and >50%), writing all interval-wise metrics to bubble_consistency_metrics.csv. Decision point: HiFiCCL-Hifiasm_graph should consistently outperform Hifiasm_graph across thresholds (e.g. precision 0.9067 / recall 0.6974 at >0% vs 0.8435 / 0.5399 at >50%).

## Related tools

- **minigraph** (Constructs the pangenome graph from GRCh38 reference plus CHM13 (v1.0) and the 45 per-sample assemblies; also built the HPRC reference graph used for comparison) — https://github.com/lh3/minigraph
- **gfatools** (Parses the constructed pangenome graph (GFA/rGFA) and emits bubble regions as a BED file for overlap-based TP/FP/FN classification) — https://github.com/lh3/gfatools
- **HiFiCCL** (Produces the HiFiCCL-Hifiasm low-coverage (~5x) assemblies whose graph is the treatment arm being evaluated) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Produces the baseline Hifiasm low-coverage assemblies whose graph serves as the comparison arm) — https://github.com/chhylp123/hifiasm

## Examples

```
./gfatools gfa2bed -m HiFiCCL-Hifiasm_graph.gfa > HiFiCCL-Hifiasm_graph.bed
```

## Evaluation signals

- True positives are defined strictly as predicted bubbles with over 80% overlap with the HPRC reference BED; verify the overlap interval used (>0% vs >50%) is recorded per row.
- Metrics must satisfy Precision = TP/(TP+FP) and Recall = TP/(TP+FN) with F1 = 2*P*R/(P+R); F1 should lie between Precision and Recall.
- Expected magnitudes at >0% threshold: precision ~0.9067, recall ~0.6974; at >50%: precision ~0.8435, recall ~0.5399 for HiFiCCL-Hifiasm_graph.
- Sanity check ordering: HiFiCCL-Hifiasm_graph precision/recall should exceed Hifiasm_graph at every threshold evaluated.
- Verify all interval-wise metrics were written to bubble_consistency_metrics.csv and that both graphs yield positive precision and recall values.

## Limitations

- Even for the better graph, recall is below 0.7 at the >0% threshold, so most reference bubbles are missed; the metric is sensitive to the 80% overlap cutoff and bubble-definition thresholds.
- At ~5x coverage, many genomic regions remain missing, so the underlying assemblies cap achievable graph recall.
- Scaffolding/reference-guided steps may overlook true SVs (only 62.5% of inversion SVs detected with RagTag scaffolding), propagating errors into the graph being scored.

## Evidence

- [methods] using the GRCh38 sequence as the reference: "using the GRCh38 sequence as the reference"
- [methods] the input data included the CHM13 (v1.0) sequence: "the input data included the CHM13 (v1.0) sequence"
- [readme] gfatools description: [evidence span withheld — anchor status: external_doc]
- [readme] minigraph incremental graph generation: [evidence span withheld — anchor status: external_doc]
- [abstract] pangenome-graph-based SV detection motivation: "improve the detection of specific germline and tumor somatic structural variants based on the pangenome graph"
