---
name: large-germline-sv-detection
description: Use when low-coverage HiFi read sets (e.g. ~5x HG002) must be evaluated
  for large germline SVs (>=10,000 bp) via pangenome graphs, or when comparing graph-based
  SV calls against HPRC_graph ground truth with precision/recall/F1.
when_to_use_negative:
- The goal is somatic SV detection on COLO829 tumor/normal data rather than germline
  SVs — that scenario uses tumor-only or tumor-normal paired minisv modes with the
  COLO829 somatic SV truth set
- The variants of interest are smaller than 10,000 bp, since this skill's evaluation
  is restricted to large germline SVs (>10,000 bp)
- No pangenome graph or HPRC_graph ground-truth alignment is available — Truvari scoring
  against ground truth cannot be performed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: minimap2
  role: Aligns HiFi reads to linear reference genomes GRCh38 and CHM13-T2T (use the
    map-hifi preset for PacBio HiFi/CCS reads)
  repo: https://github.com/lh3/minimap2
- name: minigraph
  role: Aligns the HiFi datasets to the constructed pangenome graphs (Hifiasm_graph,
    HiFiCCL-Hifiasm_graph, HPRC_graph)
  repo: https://github.com/lh3/minigraph
- name: minisv
  role: Extracts and calls SVs from linear-reference and graph alignments to produce
    per-graph SV call sets
  repo: https://github.com/lh3/minisv
- name: Truvari
  role: Compares detected SVs (v3.2.0) to the HPRC_graph ground truth and computes
    precision, recall, and F1-score metrics
- name: HiFiCCL
  role: Produces the chromosome-binned assemblies combined with hifiasm that generate
    the HiFiCCL-Hifiasm pangenome graph being evaluated
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Assembles chromosomal read bins to build the Hifiasm and HiFiCCL-Hifiasm pangenome
    graphs used in the SV detection comparison
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:e2c5a679983376de2285a7304fa8d66e64cfc1ce4480d88b23abf954ad7b1ffb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HiFiCCL
  - Truvari
  - hifiasm
  - minigraph
  - minimap2
  - minisv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# large-germline-sv-detection

## Summary

Reproduces pangenome-graph-based detection of large germline structural variants (>=10,000 bp) by aligning HiFi reads to linear references (minimap2) and pangenome graphs (minigraph), calling SVs with minisv, and scoring calls against ground truth with Truvari. It quantifies whether HiFiCCL-Hifiasm-based graphs improve germline SV detection over Hifiasm-based graphs.

## When to use

Use when low-coverage HiFi read sets (e.g. ~5x HG002) must be evaluated for large germline SVs (>=10,000 bp) via pangenome graphs, or when comparing graph-based SV calls against HPRC_graph ground truth with precision/recall/F1.

## When NOT to use

- The goal is somatic SV detection on COLO829 tumor/normal data rather than germline SVs — that scenario uses tumor-only or tumor-normal paired minisv modes with the COLO829 somatic SV truth set
- The variants of interest are smaller than 10,000 bp, since this skill's evaluation is restricted to large germline SVs (>10,000 bp)
- No pangenome graph or HPRC_graph ground-truth alignment is available — Truvari scoring against ground truth cannot be performed

## Inputs

- HiFi read datasets (e.g. HG002, SRR10382244)
- Linear reference genomes GRCh38 and CHM13-T2T (CHM13 v2.0/v1.0)
- Pangenome graphs in GFA format: Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph
- HPRC_graph-derived SV calls used as germline ground truth

## Outputs

- SV call sets per graph (minisv output on linear and graph alignments)
- Precision, recall, and F1 metrics from Truvari v3.2.0 comparison
- sv_detection_evaluation.csv with all computed metrics

## How to apply

First align the input HiFi datasets to the linear reference genomes GRCh38 and CHM13-T2T with minimap2, and align the same datasets to the constructed pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph) with minigraph. Run minisv on the two linear-reference alignments plus each graph alignment to obtain SV calls per graph. For germline SVs larger than 10,000 bp, treat the SV calls from the HPRC_graph alignment as ground truth and score Hifiasm_graph- and HiFiCCL_graph-based calls against them using Truvari v3.2.0 to compute precision, recall, and F1. Accept the reproduction when Truvari-computed precision/F1 fall within the paper-reported ranges (e.g. germline large-SV F1 up to 75.55; HG002 HiFiCCL-Hifiasm precision 57.89% at recall 91.66%, F1 70.96) and write all metrics to sv_detection_evaluation.csv.

## Related tools

- **minimap2** (Aligns HiFi reads to linear reference genomes GRCh38 and CHM13-T2T (use the map-hifi preset for PacBio HiFi/CCS reads)) — https://github.com/lh3/minimap2
- **minigraph** (Aligns the HiFi datasets to the constructed pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph)) — https://github.com/lh3/minigraph
- **minisv** (Extracts and calls SVs from linear-reference and graph alignments to produce per-graph SV call sets) — https://github.com/lh3/minisv
- **Truvari** (Compares detected SVs (v3.2.0) to the HPRC_graph ground truth and computes precision, recall, and F1-score metrics)
- **HiFiCCL** (Produces the chromosome-binned assemblies combined with hifiasm that generate the HiFiCCL-Hifiasm pangenome graph being evaluated) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Assembles chromosomal read bins to build the Hifiasm and HiFiCCL-Hifiasm pangenome graphs used in the SV detection comparison) — https://github.com/chhylp123/hifiasm

## Examples

```
k8 minisv.js e -c "k8 minisv.js" -n COLO829T test/COLO829T.hs38l.paf.gz test/COLO829T.chm13g.paf.gz | bash > COLO829T.rsv
```

## Evaluation signals

- HG002 HiFiCCL-Hifiasm_graph yields precision 57.89% vs. 55.55% for Hifiasm_graph at the same recall (91.66%), with F1 improving from 68.74 to 70.96
- Truvari-computed germline large-SV F1 falls within the paper-reported range (up to 75.55)
- All metrics are written to sv_detection_evaluation.csv with graph, precision, recall, and F1 fields populated
- Ground truth for germline SVs >10,000 bp is taken from the HPRC_graph alignment, not from a linear-reference call set
- Calls are filtered to SVs larger than 10,000 bp before Truvari comparison

## Limitations

- RagTag scaffolding of assemblies may overlook true SVs — only 62.5% of inversion SVs were detected, so graph construction upstream can miss events
- At 5x coverage many genomic regions remain missing, leaving completeness gaps that limit SV detection
- Evaluation depends on HPRC_graph calls as ground truth, which may itself be incomplete for large germline SVs

## Evidence

- [methods] Align to linear references: [evidence span withheld — anchor status: not_found]
- [methods] Ground truth definition: [evidence span withheld — anchor status: not_found]
- [results] Key quantitative finding: "HiFiCCL‐Hifiasm_graph achieved higher precision (57.89% vs. 55.55%) with the same recall (91.66%) compared to Hifiasm_graph, improving the F1 score from 68.74 to 70.96"
- [abstract] Abstract claim: "improve the detection of specific germline and tumor somatic structural variants based on the pangenome graph"
- [readme] minisv invocation: "k8 minisv.js e -c "k8 minisv.js" -n COLO829T test/COLO829T.hs38l.paf.gz
  test/COLO829T.chm13g.paf.gz | bash > COLO829T.rsv"
