---
name: sv-benchmark-against-truth-set
description: Use when a candidate SV call set (from assembly-to-reference callers
  like SVIM-asm or from minisv on pangenome graphs) must be scored against a truth
  set for large SVs (>7000 bp or ≥10000 bp) to report precision/recall/F1.
when_to_use_negative:
- The question is assembly contiguity or misassembly length (e.g., the 21.19%/38.58%
  misassembled-contig reduction metric), not SV-call benchmarking against a truth
  set
- No curated truth set exists for the sample — e.g., a plant genome (rice, Arabidopsis
  thaliana) for which only two samples were tested and no GIAB/T2T benchmark applies
- SVs smaller than the benchmark cutoff (≤7000 bp for the SVIM-asm Tier 1 comparison,
  ≤10000 bp for the germline pangenome-graph comparison) are the target
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Truvari
  role: Compares filtered SV calls to truth sets and computes precision, recall, and
    F1 (v3.2.0)
- name: SVIM-asm
  role: Calls SVs from assembly-to-reference alignments on primary+alternate assemblies
    (v1.0.3), prior to truth-set comparison
- name: minisv
  role: Extracts candidate SVs from linear (minimap2) and graph (minigraph) alignments
    feeding the benchmarked call sets
  repo: https://github.com/lh3/minisv
- name: minimap2
  role: Aligns HiFi reads to GRCh38/CHM13 and assemblies to references (asm5 intra-species
    preset) to generate comparison-space alignments
  repo: https://github.com/lh3/minimap2
- name: minigraph
  role: Aligns read sets to constructed pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph,
    HPRC_graph) whose calls are scored against HPRC_graph ground truth
  repo: https://github.com/lh3/minigraph
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:5aca9223a7e0b251b0b0365c05ab9bf2e5ea3a3aa6369da1e9df8be9a052cd1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minisv
  tools:
  - HiFiCCL
  - SVIM-asm
  - Truvari
  - minigraph
  - minimap2
  - minisv
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# sv-benchmark-against-truth-set

## Summary

Benchmark structural-variant calls derived from assemblies or pangenome graphs against curated truth sets (GIAB Tier 1, T2T-HG002-Q100, COLO829 somatic SV set, HPRC_graph calls) using Truvari v3.2.0, computing precision, recall, and F1 at size-specific cutoffs.

## When to use

Use when a candidate SV call set (from assembly-to-reference callers like SVIM-asm or from minisv on pangenome graphs) must be scored against a truth set for large SVs (>7000 bp or ≥10000 bp) to report precision/recall/F1.

## When NOT to use

- The question is assembly contiguity or misassembly length (e.g., the 21.19%/38.58% misassembled-contig reduction metric), not SV-call benchmarking against a truth set
- No curated truth set exists for the sample — e.g., a plant genome (rice, Arabidopsis thaliana) for which only two samples were tested and no GIAB/T2T benchmark applies
- SVs smaller than the benchmark cutoff (≤7000 bp for the SVIM-asm Tier 1 comparison, ≤10000 bp for the germline pangenome-graph comparison) are the target

## Inputs

- Assembly FASTAs (Hifiasm and HiFiCCL-Hifiasm primary+alternate; HiFlye/LJA variants) from Zenodo deposit DOI 10.5281/zenodo.17204429
- Assembly-to-reference alignments (minimap2) or read-to-reference/graph alignments (minimap2 to GRCh38/CHM13-T2T, minigraph to GFA pangenome graphs)
- SV call sets from SVIM-asm 1.0.3 (assembly space, filtered >7000 bp) or minisv (graph/linear alignment space)
- Truth sets: GIAB Tier 1, T2T-HG002-Q100, COLO829 somatic SV benchmark, HPRC_graph-derived germline SV calls (≥10000 bp)

## Outputs

- Truvari v3.2.0 comparison results yielding precision, recall, and F1 per call set (e.g., svim_asm_sv_benchmark_metrics.tsv)
- Aggregated evaluation CSV across graphs and modes (sv_detection_evaluation.csv) with germline, tumor-only, and tumor-normal metrics
- Size-stratified F1 values (e.g., insertion bins 10–15 kb and 15–20 kb)

## How to apply

First generate or obtain SV calls in a comparable space: for assembly-based calling, align primary+alternate assembly FASTAs to GRCh37 and run SVIM-asm (1.0.3), filtering to SVs >7000 bp; for graph-based calling, align reads to GRCh38/CHM13 with minimap2 and to the pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph) with minigraph, then run minisv. Next, treat the appropriate benchmark as ground truth — GIAB Tier 1 or T2T-HG002-Q100 for HG002 germline, HPRC_graph alignment results for germline SVs larger than 10,000 bp, and the COLO829 somatic SV truth set for tumor-only or tumor-normal (COLO829 vs COLO829BL) scenarios. Run Truvari v3.2.0 to compare the call set to the truth set, combining primary and alternate assemblies where applicable. Stratify by SV size when the paper does — insertion F1 per bin (10–15 kb: 0.5851; 15–20 kb: 0.6038) — and write precision/recall/F1 to a machine-readable table (e.g., svim_asm_sv_benchmark_metrics.tsv or sv_detection_evaluation.csv).

## Related tools

- **Truvari** (Compares filtered SV calls to truth sets and computes precision, recall, and F1 (v3.2.0))
- **SVIM-asm** (Calls SVs from assembly-to-reference alignments on primary+alternate assemblies (v1.0.3), prior to truth-set comparison)
- **minisv** (Extracts candidate SVs from linear (minimap2) and graph (minigraph) alignments feeding the benchmarked call sets) — https://github.com/lh3/minisv
- **minimap2** (Aligns HiFi reads to GRCh38/CHM13 and assemblies to references (asm5 intra-species preset) to generate comparison-space alignments) — https://github.com/lh3/minimap2
- **minigraph** (Aligns read sets to constructed pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph) whose calls are scored against HPRC_graph ground truth) — https://github.com/lh3/minigraph

## Examples

```
./minimap2 -cx asm5 GRCh37.fa assembly.p_ctg.fa > assembly2ref.paf
```

## Evaluation signals

- Tier 1 GIAB benchmark of HiFiCCL-Hifiasm SVIM-asm calls (>7000 bp) reproduces precision 0.9193, recall 0.5643, F1 0.6993
- T2T-HG002-Q100 benchmark reproduces recall 0.4369 (vs Sniffles2 0.3526), precision 0.9255, F1 0.6165 for primary+alternate assemblies
- Insertion size-stratified F1 matches 0.5851 (10–15 kb) and 0.6038 (15–20 kb)
- Pangenome-graph germline large-SV (≥10000 bp) metrics: HiFiCCL-Hifiasm_graph precision 57.89% vs 55.55% at equal recall 91.66%, F1 70.96 vs 68.74 (Hifiasm_graph); reported germline large-SV F1 range up to 75.55
- Somatic COLO829 evaluation: tumor-only precision 51.28%/F1 41.23; tumor-normal precision 95.23%/F1 50.63; HPRC_graph_plus F1 43.95

## Limitations

- RagTag-based scaffolding relies on reference alignments and may overlook true SVs — only 62.5% of inversion SVs were detected in the article's discussion
- Germline benchmark uses HPRC_graph alignment results as ground truth rather than an external curated set, so scores are relative to that proxy
- At ~5x coverage many genomic regions remain missing, so recall for large SVs is bounded by assembly completeness

## Evidence

- [methods] Compare the filtered SV calls against the GIAB Tier 1 truth set and the T2T-HG002-Q100 benchmark using Truvari (v3.2.0), with GRCh37 as the reference genome: [evidence span withheld — anchor status: not_found]
- [intro] We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp): "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp)"
- [abstract] These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [discussion] the scaffolding process relies on reference genome alignments, which may overlook true SVs: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"
- [readme] ./minimap2 -cx asm5 asm1.fa asm2.fa > aln.paf             # intra-species asm-to-asm alignment: [evidence span withheld — anchor status: external_doc]
