---
name: precision-recall-f1-evaluation-of-sv-callers
description: Use when you need to quantitatively compare SV caller outputs against
  benchmark truth sets (e.g. GIAB samples like NA19240, HG00514, HG00733) and report
  DEL/INS precision, recall and F1.
when_to_use_negative:
- You need raw call-set comparison statistics (e.g. genotype concordance or size-banded
  matches) rather than precision/recall/F1 summary of SV detection
- The input is a single method's VCF with no benchmark truth set available — no precision/recall/F1
  can be computed without a ground truth
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Consensus SV caller whose filtered VCF is one of the benchmarked methods against
    ground truth
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the pipeline producing the evaluated VCFs is run
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline runner that generates the per-sample vcfs folders containing the
    caller VCFs used in the benchmark
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Truvari
  role: Runs bench per variant type (DEL, INS) to compute precision, recall and F1
    of each method against the truth set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:1a8a9983fad31841cef6e4418cac0d31d2148c161d57d259175c5aec4aa5377e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  repo_url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - Docker
  - Nextflow
  - Truvari
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Precision, recall and F1 evaluation of SV callers with Truvari bench

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks structural variant callers (e.g. ConsensuSV-ONT, six individual ONT callers, the collapsed candidate set, and cnnLSV) against ground-truth VCFs by running Truvari bench separately for deletions (DEL) and insertions (INS), then extracting precision, recall and F1 per method, sample and variant type.

## When to use

Use when you need to quantitatively compare SV caller outputs against benchmark truth sets (e.g. GIAB samples like NA19240, HG00514, HG00733) and report DEL/INS precision, recall and F1.

## When NOT to use

- You need raw call-set comparison statistics (e.g. genotype concordance or size-banded matches) rather than precision/recall/F1 summary of SV detection
- The input is a single method's VCF with no benchmark truth set available — no precision/recall/F1 can be computed without a ground truth

## Inputs

- Per-sample VCF files of ConsensuSV-ONT consensus calls (from the vcfs output folder of the nextflow pipeline)
- Per-sample VCF files from the six individual SV callers
- Per-sample collapsed candidate VCF set
- cnnLSV VCF calls
- Benchmark ground-truth VCF sets for samples NA19240, HG00514 and HG00733 (ONT datasets)
- Variant type filter specifications for DEL and INS

## Outputs

- One Truvari bench result per method x sample x variant type (DEL, INS)
- Assembled CSV table of precision, recall and F1 organized by method, sample and variant type

## How to apply

Run Truvari bench separately for deletions and insertions, comparing each method's VCF (ConsensuSV-ONT consensus VCF, each of the six individual callers' VCFs, the collapsed candidate set from the vcfs output folder, and cnnLSV calls) against the benchmark truth set per sample. Extract precision, recall and F1 from each Truvari bench run and assemble them into one CSV table organized by method, sample and variant type. Interpret the collapsed set as a recall ceiling but expect low F1 due to low precision, while the CNN-filtered consensus should show superior precision and F1 (reported F1 for DEL NA19240 = 0.6733 and INS NA19240 = 0.6487 for ConsensuSV-ONT). Validate by checking that the assembled table reproduces these reported Table 1 values.

## Related tools

- **ConsensuSV-ONT** (Consensus SV caller whose filtered VCF is one of the benchmarked methods against ground truth) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the pipeline producing the evaluated VCFs is run) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline runner that generates the per-sample vcfs folders containing the caller VCFs used in the benchmark) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Truvari** (Runs bench per variant type (DEL, INS) to compute precision, recall and F1 of each method against the truth set)

## Examples

```
nextflow pipeline.nf --input files.csv --outdir results && truvari bench -b truth_NA19240.vcf.gz -c results/vcfs/NA19240/consensuSV-ONT.vcf --types DEL
```

## Evaluation signals

- Assembled CSV table reproduces the reported Table 1 values, e.g. F1 ConsensuSV-ONT DEL NA19240 = 0.6733 and F1 INS NA19240 = 0.6487
- ConsensuSV-ONT F1 exceeds each of the six single callers and cnnLSV for both DEL and INS in each of NA19240, HG00514, HG00733
- Collapsed set shows the highest recall but low F1 due to low precision, consistent with unfiltered candidate behavior
- One Truvari bench run exists for every combination of method x sample x variant type (DEL and INS)
- All precision/recall/F1 values are within [0, 1] and F1 equals the harmonic mean of the reported precision and recall

## Limitations

- Benchmark covers only ONT long-read data and only deletions and insertions (DEL, INS) as variant types
- Evaluation is limited to the three ONT datasets (NA19240, HG00514, HG00733); results may not transfer to other samples or technologies
- The article notes a need for a more detailed comparative analysis of mutation detection using different tools and between different sequencing methods

## Evidence

- [other] Task workflow quote on benchmarking procedure: [evidence span withheld — anchor status: not_found]
- [results] Task finding quote on performance outcome: [evidence span withheld — anchor status: not_found]
