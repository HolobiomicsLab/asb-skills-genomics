---
name: precision-recall-f1-evaluation
description: Use when SV calls (e.g., minisv calls >=10000 bp from pangenome or linear-reference
  alignments) must be quantitatively compared against a benchmark truth set to report
  precision, recall, and F1.
when_to_use_negative:
- When the goal is novel SV discovery rather than benchmarking an existing callset
  against a truth set
- When evaluating SVs shorter than the 10,000 bp threshold used for germline large-SV
  scoring
- When no accepted ground-truth callset (e.g., HPRC_graph or GIAB-style benchmark)
  is available for the sample
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Compares detected SVs to truth sets and computes precision, recall, and F1-score
    metrics (v3.2.0)
- name: minisv
  role: Extracts candidate SVs from linear-reference (minimap2) and pangenome-graph
    (minigraph) alignments; supports tumor-only mode
  repo: https://github.com/lh3/minisv
- name: minimap2
  role: Aligns HiFi reads to GRCh38 and CHM13-T2T and produces reciprocal tumor/normal
    alignments
  repo: https://github.com/lh3/minimap2
- name: minigraph
  role: Aligns the same HiFi datasets to the constructed pangenome graphs to generate
    graph-based SV calls
  repo: https://github.com/lh3/minigraph
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:c6520deddb7516274ba90147c01907f69f6372266fb7dfe93266ff137782b8c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minisv
  tools:
  - HiFiCCL
  - Truvari
  - minigraph
  - minimap2
  - minisv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# precision-recall-f1-evaluation

## Summary

Benchmark workflow for scoring SV callsets against a designated truth set using Truvari v3.2.0, computing precision, recall, and F1 to compare graph-based SV detection across pangenome and linear-reference pipelines.

## When to use

Use when SV calls (e.g., minisv calls >=10000 bp from pangenome or linear-reference alignments) must be quantitatively compared against a benchmark truth set to report precision, recall, and F1.

## When NOT to use

- When the goal is novel SV discovery rather than benchmarking an existing callset against a truth set
- When evaluating SVs shorter than the 10,000 bp threshold used for germline large-SV scoring
- When no accepted ground-truth callset (e.g., HPRC_graph or GIAB-style benchmark) is available for the sample

## Inputs

- minisv SV callsets from graph alignments (GAF-derived) for Hifiasm_graph, HiFiCCL-Hifiasm_graph, and HPRC_graph
- minisv SV callsets from minimap2 alignments to GRCh38 and CHM13-T2T
- HPRC_graph-derived germline SV calls serving as ground truth for SVs >=10000 bp
- COLO829 somatic SV truth set (tumor-only and tumor-normal scenarios)
- Paired COLO829/COLO829BL (30x) minimap2 alignments for tumor-normal calling

## Outputs

- Truvari comparison report with precision, recall, and F1 per callset
- sv_detection_evaluation.csv containing all computed metrics

## How to apply

Run minisv on alignments to linear references (GRCh38, CHM13-T2T via minimap2) and to pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph) via minigraph. For germline SVs larger than 10,000 bp, treat the HPRC_graph-based calls as ground truth and score the other callsets against it with Truvari (v3.2.0). For somatic SVs, run minisv in tumor-only mode on COLO829 (30x) alignments and evaluate against the COLO829 SV benchmark; for the tumor-normal paired scenario supplement with reciprocal minimap2 alignments between COLO829 and COLO829BL (30x) and evaluate against the COLO829 somatic SV truth set. Truvari outputs precision, recall, and F1 per comparison; accept the reproduction when the metrics fall within the paper-reported ranges and write all values to sv_detection_evaluation.csv.

## Related tools

- **Truvari** (Compares detected SVs to truth sets and computes precision, recall, and F1-score metrics (v3.2.0))
- **minisv** (Extracts candidate SVs from linear-reference (minimap2) and pangenome-graph (minigraph) alignments; supports tumor-only mode) — https://github.com/lh3/minisv
- **minimap2** (Aligns HiFi reads to GRCh38 and CHM13-T2T and produces reciprocal tumor/normal alignments) — https://github.com/lh3/minimap2
- **minigraph** (Aligns the same HiFi datasets to the constructed pangenome graphs to generate graph-based SV calls) — https://github.com/lh3/minigraph

## Examples

```
k8 minisv.js e -c "k8 minisv.js" -n COLO829T COLO829T.hs38l.paf.gz COLO829T.chm13g.paf.gz | bash > COLO829T.rsv
```

## Evaluation signals

- Truvari-computed germline large-SV F1 falls within paper-reported range (up to 75.55); HG002 HiFiCCL-Hifiasm_graph F1 = 70.96 vs Hifiasm_graph 68.74
- HG002 precision values match reported 57.89% (HiFiCCL-Hifiasm_graph) vs 55.55% (Hifiasm_graph) at identical recall 91.66%
- Tumor-only COLO829 metrics match precision 51.28% / F1 41.23; tumor-normal match precision 95.23% / F1 50.63; HPRC_graph_plus F1 = 43.95
- All metrics are written to sv_detection_evaluation.csv with one row per callset/truth-set comparison
- Only SVs >=10000 bp are included in germline large-SV scoring

## Limitations

- Evaluation for germline SVs relies on HPRC_graph-derived calls as ground truth, so accuracy is relative to that graph rather than absolute truth
- Reported F1 values (max ~70.96 for HG002, 41-50 for somatic scenarios) indicate substantial false-negative and false-positive rates at the >=10000 bp scale
- Somatic tumor-normal evaluation requires supplemental reciprocal minimap2 alignments beyond the standard minisv inputs

## Evidence

- [methods] For germline SVs larger than 10,000 bp, treat the SV results from the HPRC_graph alignment as the ground truth and score the Hifiasm_graph and HiFiCCL_graph-based calls against it.: [evidence span withheld — anchor status: not_found]
- [methods] Use Truvari (v3.2.0) to compare detected SVs to the truth sets and compute precision, recall, and F1-score metrics.: [evidence span withheld — anchor status: not_found]
- [results] HiFiCCL‐Hifiasm_graph achieved higher precision (57.89% vs. 55.55%) with the same recall (91.66%) compared to Hifiasm_graph, improving the F1 score from 68.74 to 70.96: "HiFiCCL‐Hifiasm_graph achieved higher precision (57.89% vs. 55.55%) with the same recall (91.66%) compared to Hifiasm_graph, improving the F1 score from 68.74 to 70.96"
- [methods] run minisv in tumor-only mode on the COLO829 (30×) alignments and evaluate SVs >10,000 bp against the COLO829 SV benchmark set: [evidence span withheld — anchor status: not_found]
- [methods] Run minisv on the two linear-reference alignments plus each graph alignment to obtain SV calls per graph.: [evidence span withheld — anchor status: not_found]
