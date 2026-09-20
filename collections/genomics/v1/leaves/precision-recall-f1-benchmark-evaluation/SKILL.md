---
name: precision-recall-f1-benchmark-evaluation
description: Use when a variant-calling benchmark needs standardized precision/recall/F1
  numbers for ONT callsets (e.g. HG002 k12/k14 BAMs) against a GIAB truth set with
  stratification masking.
when_to_use_negative:
- The goal is structural variant or indel calling evaluation, since miniSNV targets
  single nucleotide variant identification
- No matched GIAB truth set or stratification annotations exist for the sample genome,
  so precision/recall/F1 cannot be grounded in confident regions
- Input data are simulated ONT-like reads without a reference truth set to benchmark
  against
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: Statistical SNV caller whose callsets are benchmarked; run with known-variant
    background enabled and disabled for ablation
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Baseline deep-learning variant caller used for precision/recall/F1 comparison
    (ONT-trained model from rerio)
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Baseline diploid SNV calling method used for F1-score comparison
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the real long-read datasets (HG002 k12 and k14) benchmarked against
    the truth set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:242a21328b357536fd29d11c9eebc368564a5d7804647d7e7bf00956b6ad5dac
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
  - Oxford Nanopore Technologies (ONT)
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precision-recall-f1-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate SNV callsets from ONT variant callers (e.g. miniSNV) against the GIAB HG002 truth set using GIAB v3.3 stratifications, computing precision, recall, and F1-score to compare callers and ablation conditions.

## When to use

Use when a variant-calling benchmark needs standardized precision/recall/F1 numbers for ONT callsets (e.g. HG002 k12/k14 BAMs) against a GIAB truth set with stratification masking.

## When NOT to use

- The goal is structural variant or indel calling evaluation, since miniSNV targets single nucleotide variant identification
- No matched GIAB truth set or stratification annotations exist for the sample genome, so precision/recall/F1 cannot be grounded in confident regions
- Input data are simulated ONT-like reads without a reference truth set to benchmark against

## Inputs

- Sorted, indexed BAM files of ONT reads aligned to GRCh38 (e.g. HG002 k12-data ligation kit v12 R10.4 FLO-PRO112 simplex, k14-data Dorado basecalled)
- GIAB HG002 truth-set VCF
- GIAB v3.3 stratification annotations (difficult-to-map and low-complexity regions)
- Caller output VCFs (miniSNV with/without known-variant index, Clair3, Longshot)

## Outputs

- Per-callset precision, recall, and F1-score metrics
- Ablation/comparison table (e.g. ablation_table.csv) across dataset x background conditions
- Directional deltas versus baselines (e.g. +0.02% precision over Clair3; +0.05%/+1.83%/+0.96% precision/recall/F1 over Longshot on k12-data)

## How to apply

Align ONT reads to GRCh38 with Minimap2 and produce sorted, indexed BAMs, then run each caller (e.g. miniSNV with known-variant background enabled/disabled, plus baselines like Clair3 and Longshot) to generate callsets. Evaluate every callset against the GIAB HG002 truth set with GIAB v3.3 stratifications applied to restrict to confident regions (excluding difficult-to-map and low-complexity regions). Compute precision, recall, and F1 per callset and tabulate the dataset x condition matrix (e.g. k12 vs k14, known-variant index on/off) into a comparison table such as ablation_table.csv. Accept the result if directional effects reproduce: miniSNV plus known-variant background should exceed 99% F1 on k12-data and match published deltas over Clair3 and Longshot.

## Related tools

- **miniSNV** (Statistical SNV caller whose callsets are benchmarked; run with known-variant background enabled and disabled for ablation) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Baseline deep-learning variant caller used for precision/recall/F1 comparison (ONT-trained model from rerio)) — https://github.com/nanoporetech/rerio
- **Longshot** (Baseline diploid SNV calling method used for F1-score comparison)
- **Oxford Nanopore Technologies (ONT)** (Source of the real long-read datasets (HG002 k12 and k14) benchmarked against the truth set)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k12.sorted.bam --workDir work_k12 --useindex -i index_folder --human
```

## Evaluation signals

- miniSNV with known-variant background exceeds 99% F1-score on HG002 k12-data
- Regenerated table reproduces reported ablation deltas: +0.12% precision and +0.04% F1 on k12-data; +0.08% precision, +0.16% recall, +0.12% F1 on k14-data with known variants
- Only miniSNV exceeds 99% F1 among compared callers on k12-data, beating Longshot by 0.05%/1.83%/0.96% precision/recall/F1
- All callsets are evaluated under identical GIAB v3.3 stratification masks so difficult-to-map and low-complexity regions are consistently excluded
- Metrics are computed on all four dataset x background conditions (k12/k14 x known-variant index on/off) with identical filtering (MAPQ >= 20)

## Limitations

- miniSNV's SNV-focused pipeline does not address structural variants or indels, so benchmarking is limited to SNV identification
- Truth-set benchmarks depend on the GIAB HG002 truth set and v3.3 stratifications; regions outside these stratifications are not assessed
- Known-variant background relies on population VCF prior (1000 Genomes by default), so results may differ for populations or species lacking curated common-variant resources

## Evidence
