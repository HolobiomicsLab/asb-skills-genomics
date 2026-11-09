---
name: truvari-benchmark-vcf-comparison
description: Use when you have VCF files from one or more SV callers (plus a consensus
  method) and benchmark truth sets, and need precision, recall, and F1 computed per
  caller, sample, and variant type (DEL, INS).
when_to_use_negative:
- Input VCFs have not been collapsed and CNN-filtered yet (sigmoid output threshold
  0.5 not applied), so the consensus call set does not yet exist as a VCF
- The task requires short-read SV benchmarking or comparison of raw fastq.gz alignments
  rather than finished per-sample VCF call sets
- Only one caller's VCF and one variant type are of interest without cross-method,
  cross-sample comparison against ground truth
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Produces the consensus VCFs (collapsed candidates filtered by a 3D CNN) that
    are benchmarked against single callers, the collapsed set, and cnnLSV
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment hosting the nextflow pipeline that generates the vcfs
    folder used as Truvari bench input
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline engine that runs the six callers and consensus generation in parallel,
    writing the per-sample VCFs compared with Truvari
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:b2650a8a84e24f7dbc7738a8dce36f086a9a85fa531ee9e57404799656ef79b7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# truvari-benchmark-vcf-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks structural variant call sets against ground truth with Truvari bench, run separately per SV type, to extract precision, recall, and F1 as used to validate ConsensuSV-ONT versus six individual callers, the collapsed candidate set, and cnnLSV on NA19240, HG00514, and HG00733 ONT data.

## When to use

Use when you have VCF files from one or more SV callers (plus a consensus method) and benchmark truth sets, and need precision, recall, and F1 computed per caller, sample, and variant type (DEL, INS).

## When NOT to use

- Input VCFs have not been collapsed and CNN-filtered yet (sigmoid output threshold 0.5 not applied), so the consensus call set does not yet exist as a VCF
- The task requires short-read SV benchmarking or comparison of raw fastq.gz alignments rather than finished per-sample VCF call sets
- Only one caller's VCF and one variant type are of interest without cross-method, cross-sample comparison against ground truth

## Inputs

- Per-sample VCF files from the six individual SV callers, the collapsed candidate set, and the ConsensuSV-ONT filtered VCF (from the vcfs output folder of the nextflow pipeline)
- Benchmark truth set VCFs for NA19240, HG00514, and HG00733 ONT samples

## Outputs

- Truvari bench precision, recall, and F1 values per run (one per method x sample x SV type)
- Assembled CSV table organized by method, sample, and variant type (reproducing Table 1)

## How to apply

Run Truvari bench separately for deletions (DEL) and insertions (INS), comparing each method — ConsensuSV-ONT, each of the six individual long-read SV callers, the collapsed candidate set, and cnnLSV — against the benchmark truth sets for each sample (NA19240, HG00514, HG00733). The VCF inputs come from the vcfs output folder of the ConsensuSV-ONT nextflow pipeline, which stores per-sample folders with all VCFs from the individual SV calling tools and consensuSV-ONT; the ConsensuSV-ONT VCF itself is the collapsed candidate set filtered by a 3D CNN (keeping variants with sigmoid output > 0.5). Extract precision, recall, and F1 from each Truvari bench run and assemble them into one CSV table organized by method, sample, and variant type. Interpret with the expected pattern in mind: the collapsed set yields the highest recall but low F1 due to low precision, while the CNN-filtered consensus achieves superior precision and F1.

## Related tools

- **ConsensuSV-ONT** (Produces the consensus VCFs (collapsed candidates filtered by a 3D CNN) that are benchmarked against single callers, the collapsed set, and cnnLSV) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment hosting the nextflow pipeline that generates the vcfs folder used as Truvari bench input) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline engine that runs the six callers and consensus generation in parallel, writing the per-sample VCFs compared with Truvari) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv; truvari bench -b truth_NA19240.vcf -c vcfs/NA19240/consensuSV-ONT.vcf --svtype DEL
```

## Evaluation signals

- The assembled CSV reproduces reported Table 1 values, e.g. F1 ConsensuSV-ONT DEL NA19240 = 0.6733 and INS NA19240 = 0.6487
- The collapsed candidate set shows the highest recall but low F1 due to low precision, while ConsensuSV-ONT outperforms each single caller and cnnLSV
- Truvari bench was executed separately for DEL and INS for every method x sample combination, yielding 2 x methods x samples bench runs
- Output table rows are keyed by method, sample, and variant type with all three metrics (precision, recall, F1) populated

## Limitations

- The benchmark is limited to ONT long-read data on three GIAB-style samples (NA19240, HG00514, HG00733) and two SV types (DEL, INS); the authors note a need for a more detailed comparative analysis across tools and sequencing methods
- No changelog was found in the repository artifact, so Truvari/consensus version drift between runs may be hard to trace
- The article's methods section does not enumerate the six individual SV caller names in the reviewed excerpts, making exact per-caller reproduction dependent on the repository

## Evidence

- [other] CNN filtering threshold for consensus calls: [evidence span withheld — anchor status: not_found]
- [other] expected Table 1 performance pattern: [evidence span withheld — anchor status: not_found]
