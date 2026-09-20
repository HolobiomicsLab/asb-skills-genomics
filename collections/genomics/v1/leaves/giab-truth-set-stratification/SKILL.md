---
name: giab-truth-set-stratification
description: Use when comparing variant callers (e.g., miniSNV vs Clair3) on GIAB
  reference samples and you need stratified precision/recall/F1 rather than a single
  whole-genome accuracy number.
when_to_use_negative:
- Input is not a called VCF against a GIAB reference sample (e.g., raw reads or simulated
  data without a truth set) — stratified truth-set comparison is undefined.
- You need variant calling itself rather than caller evaluation — use the miniSNV
  or Clair3 calling workflows instead.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: Statistical SNV caller being benchmarked; its VCFs are evaluated against GIAB
    truth sets with v3.3 stratifications
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Deep-learning comparison caller; run with the latest ONT-trained model from
    the rerio clair3_models repository before truth-set evaluation
  repo: https://github.com/nanoporetech/rerio/tree/master/clair3_models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:3df400722b99b24ea1b47ad02e9d9b18ab3a1f778bcc65b009278e7902f49909
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# giab-truth-set-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SNV-caller output by evaluating each sample's VCF against Genome in a Bottle (GIAB) truth sets for the Ashkenazi Trio (HG002/HG003/HG004), applying GIAB v3.3 stratifications for difficult-to-map and low-complexity regions, and reporting precision, recall, and F1 per sample.

## When to use

Use when comparing variant callers (e.g., miniSNV vs Clair3) on GIAB reference samples and you need stratified precision/recall/F1 rather than a single whole-genome accuracy number.

## When NOT to use

- Input is not a called VCF against a GIAB reference sample (e.g., raw reads or simulated data without a truth set) — stratified truth-set comparison is undefined.
- You need variant calling itself rather than caller evaluation — use the miniSNV or Clair3 calling workflows instead.

## Inputs

- Called variant VCFs per sample (one per caller, per GIAB sample: HG002, HG003, HG004)
- GIAB truth-set VCFs for the Ashkenazi Trio
- GIAB v3.3 stratification BEDs for difficult-to-map and low-complexity regions
- Dorado hac-basecalled, Minimap2-aligned, sorted and samtools-indexed BAM files on GRCh38

## Outputs

- Per-caller x per-sample precision, recall, and F1 metrics (e.g., benchmark_table_hac.csv)
- Comparison of F1/recall/precision per GIAB sample against competing callers

## How to apply

Run each caller (e.g., miniSNV on Dorado hac-basecalled, Minimap2-aligned GRCh38 BAMs with default MAPQ >= 20 filtering, R_alt 0.2, and the 1000 Genomes common-variants index; Clair3 with the latest ONT-trained model from nanoporetech/rerio clair3_models) to obtain a per-sample VCF. Compare each VCF against the GIAB truth sets for HG002, HG003, and HG004, restricting assessment with GIAB v3.3 stratifications that exclude difficult-to-map and low-complexity regions of the genome so both callers are scored on confidently callable territory. Compute precision, recall, and F1 per sample and assemble them into a per-caller x per-sample benchmark table (e.g., benchmark_table_hac.csv). Judge success by checking that F1 scores exceed 99.7% for both callers on all three samples and that per-sample differences trace to recall/precision components (e.g., miniSNV ~0.9971 vs Clair3 0.9979 on HG002, miniSNV ahead on HG003 at 0.9971 vs 0.9970).

## Related tools

- **miniSNV** (Statistical SNV caller being benchmarked; its VCFs are evaluated against GIAB truth sets with v3.3 stratifications) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Deep-learning comparison caller; run with the latest ONT-trained model from the rerio clair3_models repository before truth-set evaluation) — https://github.com/nanoporetech/rerio/tree/master/clair3_models

## Evaluation signals

- Both callers reproduce F1 scores above 99.7% on all three GIAB samples (HG002, HG003, HG004) after stratification.
- Per-sample metric tables decompose into precision, recall, and F1 components that explain ranking differences (e.g., miniSNV's HG003 edge comes from higher recall).
- Metrics are computed only within confident regions after applying GIAB v3.3 difficult-to-map and low-complexity stratifications.
- Expected ordering check: miniSNV ~0.9971 vs Clair3 0.9979 on HG002 (miniSNV lower), competitive on HG003/HG004.
- Input BAMs were samtools-indexed before calling, since callers require indexed alignment input.

## Limitations

- Scores are conditional on the GIAB v3.3 stratifications; excluding difficult-to-map and low-complexity regions hides caller differences in repetitive areas.
- Results are specific to the v14 / R10.4.1 PromethION chemistry and Dorado hac basecalling; other error profiles may change caller rankings.
- Clair3 models from rerio are research-release models compatible with Clair3 v1.x but not v2.x without conversion, so model/basecaller version mismatches can shift benchmark numbers.

## Evidence

- [results] benchmark dataset description: [evidence span withheld — anchor status: not_found]
