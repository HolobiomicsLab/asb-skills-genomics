---
name: pangenome-graph-construction
description: Use when you need to build a pangenome graph from a collection of de
  novo assemblies (GFA/FASTA) or evaluate whether one set of assemblies yields a more
  accurate graph (precision/recall of bubbles) than another.
when_to_use_negative:
- Input is a single reference genome without multiple per-sample assemblies — a pangenome
  graph requires a collection of haplotype/assembly sequences
- Your goal is short-read variant calling against an existing graph rather than building
  or benchmarking a new graph from assemblies
- Graph nodes/paths lack chromosome annotations needed for reference-guided classification
  (e.g., constructing the Pangenome Label Graph is a separate skill)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3173
tools:
- name: minigraph
  role: Constructs the pangenome graph (GFA) from the reference GRCh38 sequence plus
    CHM13 and per-sample assemblies; also maps reads to graphs with '-cx lr' for SV
    detection
  repo: https://github.com/lh3/minigraph
- name: gfatools
  role: Parses the constructed pangenome graph GFA and emits bubble regions as a BED
    file for consistency evaluation
  repo: https://github.com/lh3/gfatools
- name: HiFiCCL
  role: Produces the improved low-coverage (~5x) HiFi assemblies whose graph is compared
    against the Hifiasm-based graph
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Baseline assembler whose assemblies are used to build the comparison graph
  repo: https://github.com/chhylp123/hifiasm
- name: minisv
  role: Calls germline and somatic SVs from alignments to the constructed pangenome
    graphs for downstream validation
  repo: lh3/minisv
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:4f65b2330d882cf05de413f12fa43b368fde9067c8052043ed0249a3a044355e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minigraph
  tools:
  - HiFiCCL
  - gfatools
  - hifiasm
  - minigraph
  - minimap2
  - minisv
  - pysam
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pangenome-graph-construction

## Summary

Constructs pangenome graphs with minigraph from multiple genome assemblies (e.g., CHM13 plus 45 per-sample HiFiCCL-Hifiasm assemblies at ~5x coverage) using GRCh38 as the reference, then extracts bubble regions with gfatools to benchmark graph accuracy against an HPRC-derived reference graph.

## When to use

Use when you need to build a pangenome graph from a collection of de novo assemblies (GFA/FASTA) or evaluate whether one set of assemblies yields a more accurate graph (precision/recall of bubbles) than another.

## When NOT to use

- Input is a single reference genome without multiple per-sample assemblies — a pangenome graph requires a collection of haplotype/assembly sequences
- Your goal is short-read variant calling against an existing graph rather than building or benchmarking a new graph from assemblies
- Graph nodes/paths lack chromosome annotations needed for reference-guided classification (e.g., constructing the Pangenome Label Graph is a separate skill)

## Inputs

- Per-sample genome assemblies (FASTA) from HiFiCCL-Hifiasm and Hifiasm of 45 low-coverage (~5x) human samples, with unique sequence names
- Reference sequence GRCh38 (used as minigraph reference)
- CHM13 (v1.0) sequence to include as graph input
- BED file of bubbles from the HPRC minigraph release graph (ground truth for evaluation)

## Outputs

- Pangenome graph in GFA format per assembler variant
- BED file of bubble regions extracted by gfatools from each graph
- bubble_consistency_metrics.csv with interval-wise precision, recall, and F1 metrics
- Optionally, GAF alignments of reads to the pangenome graph for downstream SV calling

## How to apply

Rename each assembly sequence to ensure uniqueness across samples, then run minigraph with the GRCh38 sequence as the reference and the CHM13 (v1.0) sequence plus per-sample assemblies (sorted alphabetically) as input. Run gfatools on the resulting GFA to identify bubble regions and emit them as a BED file. Compare predicted bubbles against a reference BED file (from the HPRC release graph) at an 80% overlap threshold to classify true positives, false positives, and false negatives, and compute Precision = TP/(TP+FP) and Recall = TP/(TP+FN). The resulting graphs can also feed downstream SV detection: align reads with minigraph '-cx lr' and call SVs with minisv against HPRC-graph-based calls as ground truth.

## Related tools

- **minigraph** (Constructs the pangenome graph (GFA) from the reference GRCh38 sequence plus CHM13 and per-sample assemblies; also maps reads to graphs with '-cx lr' for SV detection) — https://github.com/lh3/minigraph
- **gfatools** (Parses the constructed pangenome graph GFA and emits bubble regions as a BED file for consistency evaluation) — https://github.com/lh3/gfatools
- **HiFiCCL** (Produces the improved low-coverage (~5x) HiFi assemblies whose graph is compared against the Hifiasm-based graph) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Baseline assembler whose assemblies are used to build the comparison graph) — https://github.com/chhylp123/hifiasm
- **minisv** (Calls germline and somatic SVs from alignments to the constructed pangenome graphs for downstream validation) — lh3/minisv

## Examples

```
./minigraph -cxggs GRCh38.fa CHM13.fa sample01.asm.fa sample02.asm.fa ... > pangenome.gfa && ./gfatools gfa2bed -m pangenome.gfa > bubbles.bed
```

## Evaluation signals

- HiFiCCL-Hifiasm_graph should show overall precision ~0.9067 and recall ~0.6974 at the >0% bubble-overlap threshold, and precision ~0.8435 with recall ~0.5399 at the >50% threshold, outperforming Hifiasm_graph at all thresholds
- Predicted bubbles with >80% overlap to the HPRC-graph BED count as true positives; both graphs must yield positive precision and recall values
- Interval-wise metrics (across different overlap intervals) should be written to bubble_consistency_metrics.csv
- For downstream SV validation on HG002, HiFiCCL-Hifiasm_graph-based large germline SV calls (>=10000 bp) should reach higher precision (57.89% vs 55.55%) with the same recall (91.66%) as Hifiasm_graph, F1 70.96 vs 68.74 (Truvari v3.2.0)

## Limitations

- Bubble overlap thresholds (>0%, >50%, 80% TP cutoff) are relaxed/stratified, so absolute accuracy values depend on the chosen interval
- Graph quality depends on assembly quality at low coverage (~5x), where many genomic regions remain missing due to low coverage
- Version numbers for HiFiCCL, Hifiasm, and RagTag are not stated, complicating exact reproduction

## Evidence

- [results] At relaxed bubble coverage thresholds, HiFiCCL-Hifiasm_graph achieved an overall precision of 0.9067 and recall of 0.6974 (>0%), and precision of 0.8435 with recall of 0.5399 (>50%): [evidence span withheld — anchor status: not_found]
- [methods] build the pangenome graph with minigraph, using the GRCh38 sequence as the reference: [evidence span withheld — anchor status: not_found]
- [methods] Run gfatools on each constructed pangenome graph to identify bubble regions and emit them as a BED file: [evidence span withheld — anchor status: not_found]
- [methods] classifying bubbles with over 80% overlap with the reference as true positives: [evidence span withheld — anchor status: not_found]
- [results] HiFiCCL‐Hifiasm_graph achieved higher precision (57.89% vs. 55.55%) with the same recall (91.66%): "For the HG002 dataset, HiFiCCL‐Hifiasm_graph achieved higher precision (57.89% vs. 55.55%) with the same recall (91.66%) compared to Hifiasm_graph, improving the F1 score from 68.74 to 70.96"
- [abstract] Tested on 45 human datasets (∼5× coverage), HiFiCCL combined with hifiasm reduces the length of misassembled contigs: "Tested on 45 human datasets (∼5× coverage), HiFiCCL combined with hifiasm reduces the length of misassembled contigs"
