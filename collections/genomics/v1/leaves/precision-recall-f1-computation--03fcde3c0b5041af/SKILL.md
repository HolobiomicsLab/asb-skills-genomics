---
name: precision-recall-f1-computation
description: Use when benchmarking SNV-calling VCFs (miniSNV, Clair3, Longshot) against
  GIAB truth sets for HG002/HG003/HG004 ONT data and you need per-sample precision,
  recall, and F1 within GIAB v3.3 stratification regions.
when_to_use_negative:
- The input is raw signal/POD5 or unaligned FASTQ rather than a called VCF to compare
  against a truth set
- The goal is de novo variant discovery without a GIAB truth VCF and confident-region
  stratifications to score against
- The sample is not human GIAB-truthed material (e.g., simulated Mouse/Drosophila/Arabidopsis
  ONT-like data), where GIAB truth sets and v3.3 stratifications do not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Caller whose SNV VCF output is benchmarked; reproduces F1 ~0.9971 and recall
    >99.7% on the sup-datasets
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparison caller (v1.0.4, ONT-trained model from nanoporetech/rerio clair3_models)
    whose VCFs are scored against the same GIAB truth sets
  repo: github.com/nanoporetech/rerio/tree/master/clair3_models
- name: Longshot
  role: Diploid SNV comparison caller (0.4.5) scored in the sup-dataset benchmark
    alongside miniSNV and Clair3
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:0264bf0075e825fa547d74facff33fc9789f470b70983946175065ff3a36601a
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
  - Longshot
  - miniSNV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# precision-recall-f1-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark computation for SNV callers: compare each caller's VCF against GIAB truth VCFs within GIAB v3.3 stratification regions and compute per-caller, per-sample precision, recall, and F1. Used in the miniSNV paper to evaluate miniSNV against Clair3 and Longshot on Dorado hac/sup-basecalled GIAB Ashkenazi Trio (HG002/HG003/HG004) ONT data.

## When to use

Use when benchmarking SNV-calling VCFs (miniSNV, Clair3, Longshot) against GIAB truth sets for HG002/HG003/HG004 ONT data and you need per-sample precision, recall, and F1 within GIAB v3.3 stratification regions.

## When NOT to use

- The input is raw signal/POD5 or unaligned FASTQ rather than a called VCF to compare against a truth set
- The goal is de novo variant discovery without a GIAB truth VCF and confident-region stratifications to score against
- The sample is not human GIAB-truthed material (e.g., simulated Mouse/Drosophila/Arabidopsis ONT-like data), where GIAB truth sets and v3.3 stratifications do not apply

## Inputs

- Per-caller VCF files (miniSNV, Clair3, Longshot) for HG002, HG003, HG004
- GIAB truth VCFs for the Ashkenazi Trio
- GIAB v3.3 stratification BEDs (difficult-to-map and low-complexity regions)
- Sorted GRCh38 (GCA_000001405.15) BAMs from Dorado hac/sup-basecalled ONT kit v14 R10.4.1 data

## Outputs

- Per-caller x per-sample precision, recall, and F1 metrics (e.g., benchmark_table_hac.csv, benchmark_table_sup.csv)
- Homozygous discovery rate (HDR) per caller for the sup-dataset benchmark

## How to apply

Align each caller's output (e.g., miniSNV with its 1000 Genomes prebuilt common-variants index, Clair3 v1.0.4 with the nanoporetech/rerio ONT-trained model, or Longshot 0.4.5) to the GIAB truth VCFs, restricting evaluation to the GIAB v3.3 stratifications for difficult-to-map and low-complexity regions. Compute precision, recall, and F1 per caller and per sample (HG002, HG003, HG004) for both the hac-basecalled and sup-basecalled kit v14 datasets, and assemble them into per-caller x per-sample tables (e.g., benchmark_table_hac.csv, benchmark_table_sup.csv). For the sup benchmark, additionally compute the homozygous discovery rate (HDR) per caller to quantify hom-variant recovery (miniSNV leads Clair3 by ~1.8% and Longshot by ~1.6% on HG003). Validate by checking that miniSNV's F1-score and recall exceed 99.7% on HG002/HG003/HG004 sup-datasets, and that on hac-datasets F1 reaches ~0.9971 for miniSNV versus 0.9979 for Clair3 on HG002, and 0.9971 versus 0.9970 on HG003.

## Related tools

- **miniSNV** (Caller whose SNV VCF output is benchmarked; reproduces F1 ~0.9971 and recall >99.7% on the sup-datasets) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparison caller (v1.0.4, ONT-trained model from nanoporetech/rerio clair3_models) whose VCFs are scored against the same GIAB truth sets) — github.com/nanoporetech/rerio/tree/master/clair3_models
- **Longshot** (Diploid SNV comparison caller (0.4.5) scored in the sup-dataset benchmark alongside miniSNV and Clair3)

## Evaluation signals

- miniSNV F1-score and recall exceed 99.7% on HG002, HG003, and HG004 sup-datasets
- On hac-datasets, miniSNV F1 ~0.9971 on HG002 is marginally below Clair3 (0.9979), while miniSNV edges Clair3 on HG003 (0.9971 vs 0.9970)
- Sup-dataset HDR on HG003 shows miniSNV ahead of Clair3 by ~1.8% and Longshot by ~1.6%
- All comparisons are computed within GIAB v3.3 difficult-to-map and low-complexity stratification regions, not genome-wide
- Result tables are per-caller x per-sample (3 callers x 3 trio samples for sup; miniSNV vs Clair3 x 3 samples for hac)

## Limitations

- Evaluation depends on GIAB truth VCFs and GIAB v3.3 stratifications; regions outside these annotations are excluded from the comparison
- F1 differences between callers on hac-datasets are marginal (e.g., 0.9971 vs 0.9970 on HG003), so small pipeline variations can flip the ranking
- Clair3 models from nanoporetech/rerio are research releases that Oxford Nanopore has not validated and cannot support per the README disclaimer

## Evidence

- [other] Compare caller VCFs against GIAB truth sets with stratifications, computing precision, recall, and F1 per sample: [evidence span withheld — anchor status: not_found]
- [readme] Rerio Clair3 models are unvalidated research releases: [evidence span withheld — anchor status: external_doc]
