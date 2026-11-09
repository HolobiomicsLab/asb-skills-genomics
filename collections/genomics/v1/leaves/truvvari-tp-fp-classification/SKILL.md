---
name: truvvari-tp-fp-classification
description: Use when a collapsed SV candidate VCF must be converted into labelled
  TP/FP training examples against a benchmark truth set (e.g. HGSVC) so a supervised
  CNN variant filter can be trained or evaluated.
when_to_use_negative:
- The goal is de novo variant discovery without a matched high-confidence benchmark
  VCF — Truvari bench requires a truth set to assign TP/FP labels
- Input variants are short indels/SNPs rather than structural variants; Truvari bench
  default matching thresholds are calibrated for SV calling comparison
- You need to re-run or tune the SV callers themselves — this skill only labels the
  already-collapsed consensus VCF
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: Truvari
  role: Runs 'truvari bench' with default bp and sizefold similarity thresholds to
    partition candidate SVs into TP (counterpart in HGSVC ground truth) and FP (no
    counterpart)
- name: ConsensuSV-ONT
  role: Produces the collapsed consensus candidate VCFs in the vcfs output folder
    that are the input to the TP/FP labelling step
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Pipeline engine within which the sample consensus VCFs are generated prior
    to labelling
- name: Docker
  role: Runtime environment image providing the pipeline tools (docker pull antonipietryga/consensussv-ont-pipeline)
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:77cee4a4d593902140d06d18443420b933af3787b38f23b42f833bca49a7cb97
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - Docker
  - Nextflow
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# truvvari-tp-fp-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Labels each candidate structural variant from the ConsensuSV-ONT collapsed consensus VCF as TP or FP by benchmarking it against the HGSVC ground-truth SV VCF (Chaisson et al. 2019) with Truvari bench, producing labelled training data for the CNN classification model.

## When to use

Use when a collapsed SV candidate VCF must be converted into labelled TP/FP training examples against a benchmark truth set (e.g. HGSVC) so a supervised CNN variant filter can be trained or evaluated.

## When NOT to use

- The goal is de novo variant discovery without a matched high-confidence benchmark VCF — Truvari bench requires a truth set to assign TP/FP labels
- Input variants are short indels/SNPs rather than structural variants; Truvari bench default matching thresholds are calibrated for SV calling comparison
- You need to re-run or tune the SV callers themselves — this skill only labels the already-collapsed consensus VCF

## Inputs

- Collapsed consensus SV VCF per sample from the ConsensuSV-ONT pipeline's vcfs output folder
- HGSVC benchmark SV VCF (Chaisson et al. 2019) matching the sample's genome (e.g. HG00733, HG00514)

## Outputs

- Truvari bench output directory per sample
- Per-sample labelled table tp_fp_labels.tsv with columns: sample ID, variant coordinates, SV type, TP/FP label

## How to apply

Load each sample's collapsed consensus VCF from the pipeline's 'vcfs' output folder, which holds per-sample VCFs from the individual SV callers plus the ConsensuSV-ONT consensus call. Download the matching HGSVC benchmark SV VCF for the same genome (e.g. HG00733, HG00514). Run Truvari bench with its default matching parameters (default base-distance and sizefold similarity thresholds) to compare candidate variants against the benchmark set; the rationale is that Truvari's reciprocal-match logic decides whether a candidate has a counterpart in the high-confidence long-read truth set. Parse the bench output and assign each candidate a TP label (counterpart present in the benchmark) or FP label (no counterpart). Write a per-sample tp_fp_labels.tsv with sample ID, variant coordinates, SV type, and the TP/FP label.

## Related tools

- **Truvari** (Runs 'truvari bench' with default bp and sizefold similarity thresholds to partition candidate SVs into TP (counterpart in HGSVC ground truth) and FP (no counterpart))
- **ConsensuSV-ONT** (Produces the collapsed consensus candidate VCFs in the vcfs output folder that are the input to the TP/FP labelling step) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Pipeline engine within which the sample consensus VCFs are generated prior to labelling)
- **Docker** (Runtime environment image providing the pipeline tools (docker pull antonipietryga/consensussv-ont-pipeline)) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
truvari bench -b HG00733_HGSVC_truth.vcf.gz -c HG00733_consensus.vcf.gz -o bench_HG00733  # default bp/sizefold matching; parse bench output into tp_fp_labels.tsv
```

## Evaluation signals

- tp_fp_labels.tsv exists for every processed sample
- Every candidate variant row carries exactly one TP or FP label (no missing/unlabelled entries)
- No candidate rows are dropped: row count of tp_fp_labels.tsv equals the variant count in the input collapsed VCF
- Variant coordinates and SV types in the labelled table match the corresponding entries in the input VCF
- Truvari bench runs used default matching parameters (default bp and sizefold similarity thresholds) and the correct sample-matched HGSVC benchmark VCF

## Limitations

- Label quality depends entirely on the completeness of the HGSVC benchmark set — candidates absent from the truth set are labelled FP even if they are real
- Default Truvari matching parameters (bp distance, sizefold similarity) are not tuned per sample; edge-case SVs may be mislabelled
- The article/README provide no version numbers or parameter details for the benchmarking step in this section

## Evidence

- [other] Truvari bench with default parameters divides the SV candidate set into True Positives (variants with counterparts in the ground truth set) and False Positives (variants without counterparts in the ground truth set).: [evidence span withheld — anchor status: not_found]
- [other] Run Truvari bench with default matching parameters (default bp and sizefold similarity thresholds) to compare candidate variants against the benchmark set.: [evidence span withheld — anchor status: not_found]
