---
name: long-read-sv-consensus-evaluation
description: Use when a collapsed consensus SV VCF (e.g. from ConsensuSV-ONT's vcfs
  output folder) must be labelled TP/FP against an HGSVC benchmark VCF to train or
  evaluate the CNN variant filter.
when_to_use_negative:
- Input is raw read alignments (fastq.gz) or per-caller VCFs rather than the collapsed
  consensus VCF — upstream calling/consensus stages handle those, not TP/FP labelling.
- No matching HGSVC benchmark VCF exists for the sample's genome — TP/FP labels cannot
  be assigned against Chaisson et al. 2019 truth sets.
- The goal is de novo SV discovery in a genome without a high-quality truth set, rather
  than benchmarking candidate calls against HGSVC.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Truvari
  role: Runs truvari bench with default matching parameters (default bp and sizefold
    similarity thresholds) to compare candidate variants against the HGSVC benchmark
    and classify each as TP or FP
- name: ConsensuSV-ONT
  role: Produces the collapsed consensus SV VCFs in the vcfs output folder that are
    labelled TP/FP by this skill
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment in which the pipeline (and the vcfs folder consumed by
    the labelling step) is executed
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:441079569e903765741501a1942ac2d5a8a89413bae4ce611c2e46c6e20ee79a
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
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# long-read-sv-consensus-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the collapsed consensus SV candidate VCFs produced by ConsensuSV-ONT against HGSVC ground truth (Chaisson et al. 2019) using Truvari bench, labelling each candidate True Positive or False Positive. This creates the labelled training data required for the pipeline's CNN-based filtering of high-quality variants.

## When to use

Use when a collapsed consensus SV VCF (e.g. from ConsensuSV-ONT's vcfs output folder) must be labelled TP/FP against an HGSVC benchmark VCF to train or evaluate the CNN variant filter.

## When NOT to use

- Input is raw read alignments (fastq.gz) or per-caller VCFs rather than the collapsed consensus VCF — upstream calling/consensus stages handle those, not TP/FP labelling.
- No matching HGSVC benchmark VCF exists for the sample's genome — TP/FP labels cannot be assigned against Chaisson et al. 2019 truth sets.
- The goal is de novo SV discovery in a genome without a high-quality truth set, rather than benchmarking candidate calls against HGSVC.

## Inputs

- Collapsed consensus SV VCF for each sample, taken from the pipeline's vcfs output folder
- Matching HGSVC benchmark SV VCF (Chaisson et al. 2019) for the sample's genome (e.g. HG00733, HG00514)
- Sample list csv with the path to each fastq.gz file (used to map samples to genomes)

## Outputs

- Per-sample tp_fp_labels.tsv with columns for sample ID, variant coordinates, SV type, and the TP/FP label
- Truvari bench output parsed into per-candidate TP/FP classifications for every candidate variant

## How to apply

Load each sample's collapsed consensus VCF from the pipeline's vcfs output folder (the README states this folder contains 'all VCF files from the individual SV calling tools and consensuSV-ONT'). Download the matching HGSVC benchmark SV VCF (Chaisson et al. 2019) for that sample's genome, e.g. HG00733 or HG00514 from the sample csv. Run Truvari bench with default matching parameters — its default bp overlap and sizefold similarity thresholds — so the candidate set is divided into True Positives (variants with counterparts in the ground truth set) and False Positives (variants without counterparts). Parse the bench output and write a per-sample tp_fp_labels.tsv with columns for sample ID, variant coordinates, SV type, and the TP/FP label. Validate by checking that every candidate variant carries a TP or FP label and that no candidate rows are dropped relative to the input VCF.

## Related tools

- **Truvari** (Runs truvari bench with default matching parameters (default bp and sizefold similarity thresholds) to compare candidate variants against the HGSVC benchmark and classify each as TP or FP)
- **ConsensuSV-ONT** (Produces the collapsed consensus SV VCFs in the vcfs output folder that are labelled TP/FP by this skill) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment in which the pipeline (and the vcfs folder consumed by the labelling step) is executed) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
truvari bench -b HGSVC.HG00733.sv.vcf.gz -c vcfs/HG00733/consensus.vcf.gz -o bench_HG00733/
```

## Evaluation signals

- tp_fp_labels.tsv exists for each processed sample
- Every candidate variant in tp_fp_labels.tsv carries a TP or FP label
- No candidate rows are dropped relative to the input consensus VCF (row-count invariant between input VCF and label table)
- Each TP/FP assignment reproduces Truvari bench's default-parameter partition of the candidate set against the HGSVC ground truth
- Each row contains the required columns: sample ID, variant coordinates, SV type, and TP/FP label

## Limitations

- Ground truth is restricted to HGSVC benchmark VCFs (Chaisson et al. 2019); samples whose genome lacks a benchmark cannot be labelled.
- TP/FP assignment depends on Truvari's default bp overlap and sizefold similarity thresholds; candidate/benchmark pairs just outside these defaults will be labelled FP.
- The article notes 'there is a need for a more detailed comparative analysis of mutation detection using different tools and between different sequencing methods', so labels from a single benchmark/tool combination should not be treated as universally comparable.

## Evidence

- [other] Truvari bench with default parameters divides the SV candidate set into True Positives (variants with counterparts in the ground truth set): [evidence span withheld — anchor status: not_found]
