---
name: bed-coordinate-overlap-analysis
description: Use when comparing pangenome graph constructions (e.g., from 45 low-coverage
  ~5x human assemblies) against a reference graph, where bubble regions can be emitted
  as BED and overlap-based TP/FP/FN classification is needed.
when_to_use_negative:
- The evaluation target is per-sample assembly contiguity or misassembly length (e.g.,
  HiFiCCL-Hifiasm vs Hifiasm contig QC) rather than graph-level bubble agreement with
  an existing reference graph.
- No reference pangenome graph BED exists to compare against — precision/recall under
  the 80%-overlap rule cannot be defined.
- The graph regions of interest are not bubbles (e.g., per-sample path SV calling),
  where tools like minigraph's -c calling mode or minisv are the intended route.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: gfatools
  role: Extracts bubble regions from the minigraph GFA and emits them as BED for overlap
    comparison
  repo: https://github.com/lh3/gfatools
- name: minigraph
  role: Constructs the candidate and HPRC-reference pangenome graphs whose bubbles
    are converted to BED
  repo: https://github.com/lh3/minigraph
- name: hifiasm
  role: Produces the low-coverage per-sample assemblies (HiFiCCL-Hifiasm vs Hifiasm
    variants) feeding the graphs being compared
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:503648949b2e9558e7a6f17072a966eee3660d2467725ae6c55ed37b2f3a3644
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/gfatools
  tools:
  - HiFiCCL
  - gfatools
  - hifiasm
  - minigraph
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bed-coordinate-overlap-analysis

## Summary

Evaluates pangenome graph fidelity by converting minigraph-built graphs into bubble-region BED files with gfatools and classifying predicted bubbles against an HPRC reference-graph BED via an 80% reciprocal-overlap rule to compute precision, recall, and F1.

## When to use

Use when comparing pangenome graph constructions (e.g., from 45 low-coverage ~5x human assemblies) against a reference graph, where bubble regions can be emitted as BED and overlap-based TP/FP/FN classification is needed.

## When NOT to use

- The evaluation target is per-sample assembly contiguity or misassembly length (e.g., HiFiCCL-Hifiasm vs Hifiasm contig QC) rather than graph-level bubble agreement with an existing reference graph.
- No reference pangenome graph BED exists to compare against — precision/recall under the 80%-overlap rule cannot be defined.
- The graph regions of interest are not bubbles (e.g., per-sample path SV calling), where tools like minigraph's -c calling mode or minisv are the intended route.

## Inputs

- Pangenome graph in GFA format constructed by minigraph (candidate assemblies + CHM13 v1.0 against GRCh38)
- BED file of bubble regions extracted by gfatools from the candidate graph
- BED file of bubble regions extracted from the HPRC-release reference pangenome graph

## Outputs

- Per-graph bubble BED files (from gfatools gfa2bed)
- TP/FP/FN classification per predicted bubble under the 80%-overlap rule
- Precision, recall, and F1 per overlap interval (e.g., >0%: 0.9067/0.6974; >50%: 0.8435/0.5399)
- bubble_consistency_metrics.csv with interval-wise metrics

## How to apply

Build each candidate pangenome graph with minigraph using GRCh38 as the reference and CHM13 (v1.0) plus the 45 per-sample assemblies as alphabetically sorted input, then run gfatools (gfa2bed) on the resulting GFA to emit bubble regions as a BED file. Do the same for the HPRC-release minigraph graph to obtain the reference bubble BED. Classify each predicted bubble with over 80% overlap to a reference bubble as true positive, non-overlapping predicted bubbles as false positives, and reference bubbles lacking 80% overlap as false negatives. Compute Precision = TP/(TP+FP), Recall = TP/(TP+FN), and F1 = 2xPrecisionxRecall/(Precision+Recall), repeating the evaluation at different bubble-coverage overlap intervals (e.g., >0% and >50%) and writing interval-wise metrics to bubble_consistency_metrics.csv.

## Related tools

- **gfatools** (Extracts bubble regions from the minigraph GFA and emits them as BED for overlap comparison) — https://github.com/lh3/gfatools
- **minigraph** (Constructs the candidate and HPRC-reference pangenome graphs whose bubbles are converted to BED) — https://github.com/lh3/minigraph
- **hifiasm** (Produces the low-coverage per-sample assemblies (HiFiCCL-Hifiasm vs Hifiasm variants) feeding the graphs being compared) — https://github.com/chhylp123/hifiasm

## Examples

```
./gfatools gfa2bed -m HiFiCCL-Hifiasm_graph.gfa > HiFiCCL-Hifiasm_graph.bed
```

## Evaluation signals

- Both precision and recall are positive for every overlap threshold tested (reported >0%: precision 0.9067, recall 0.6974; >50%: precision 0.8435, recall 0.5399).
- Every TP classification is backed by >80% BED-interval overlap with a reference bubble; FPs are non-overlapping, FNs are reference bubbles lacking 80% overlap.
- F1 = 2xPrecisionxRecall/(Precision+Recall) is consistent with the reported precision/recall pair for each threshold row in bubble_consistency_metrics.csv.
- BED files are reproducibly derived from GFAs via gfatools gfa2bed (bubble coordinates traceable to the minigraph GFA nodes).
- HiFiCCL-Hifiasm_graph metrics consistently exceed Hifiasm_graph metrics at all overlap intervals, matching the article's reported ranking.

## Limitations

- Assemblies at ~5x coverage leave many genomic regions missing, so recall is capped well below 1 (max reported 0.6974 at the >0% threshold).
- The 80%-overlap cutoff makes results sensitive to bubble-boundary differences; metrics shift substantially between the >0% and >50% coverage intervals.
- The comparison depends on sequence naming uniqueness across all 45 assemblies and alphabetical sorting of minigraph inputs, both required before graph construction.

## Evidence

- [methods] Compare each BED file against the BED file generated from the pangenome graph constructed by minigraph from the HPRC release: [evidence span withheld — anchor status: not_found]
- [methods] classifying bubbles with over 80% overlap with the reference as true positives (TP): [evidence span withheld — anchor status: not_found]
- [results] HiFiCCL-Hifiasm_graph achieved an overall precision of 0.9067 and recall of 0.6974: [evidence span withheld — anchor status: not_found]
- [readme] ./gfatools gfa2bed -m test/MT.gfa > MT.bed: [evidence span withheld — anchor status: external_doc]
- [readme] gfatools is a set of tools for manipulating sequence graphs in the GFA or the rGFA format: [evidence span withheld — anchor status: external_doc]
