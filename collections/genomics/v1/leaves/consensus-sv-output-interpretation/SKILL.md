---
name: consensus-sv-output-interpretation
description: Use when a ConsensuSV-ONT run has finished (vcfs/ and alignments/ folders
  exist) and you need to inspect the consensus deletion/insertion VCFs or explain
  why specific calls were scored as false positives against a benchmark set.
when_to_use_negative:
- Before the pipeline has completed — no vcfs/<sample>/ folder with caller and ConsensuSV-ONT
  VCFs exists yet
- For fresh SV calling from FASTQ data; this skill interprets existing consensus output,
  use the full ConsensuSV-ONT nextflow pipeline instead
- For short-read or Illumina-only consensus calling — the method is built for Oxford
  Nanopore long-read data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Produces the consensus SV VCFs (final CNN-filtered calls plus six individual
    caller VCFs) whose false positives this skill interprets
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the pipeline that generated the VCFs is executed
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline engine invoked inside the container (`nextflow pipeline.nf --input
    files.csv`) that emits the alignments/ and vcfs/ directories consumed by this
    skill
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:4f44109aa1fc407d1efc57ad651ab22af226d257cb548cfd93c9c7e510401283
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# consensus-sv-output-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpretation of ConsensuSV-ONT consensus structural variant calls: reading the per-sample VCFs emitted into the pipeline's 'vcfs' output directory (individual caller VCFs plus the final ConsensuSV-ONT consensus calls), and characterising consensus deletions against a ground-truth set by classifying mismatches (merged, partial-overlap, or novel false positives).

## When to use

Use when a ConsensuSV-ONT run has finished (vcfs/ and alignments/ folders exist) and you need to inspect the consensus deletion/insertion VCFs or explain why specific calls were scored as false positives against a benchmark set.

## When NOT to use

- Before the pipeline has completed — no vcfs/<sample>/ folder with caller and ConsensuSV-ONT VCFs exists yet
- For fresh SV calling from FASTQ data; this skill interprets existing consensus output, use the full ConsensuSV-ONT nextflow pipeline instead
- For short-read or Illumina-only consensus calling — the method is built for Oxford Nanopore long-read data

## Inputs

- ConsensuSV-ONT consensus deletion VCF for a sample (e.g. HG00514), from the pipeline's vcfs/<sample>/ output folder
- Ground-truth structural variant set (VCF/BED intervals) for the same sample
- Optionally, VCFs from the six individual SV callers in the same vcfs/<sample>/ folder for comparison

## Outputs

- fp_classification.csv with one row per false positive: coordinates, assigned class (merged / partial-overlap / novel), and list of overlapping ground-truth regions
- Per-call interpretation table linking consensus VCF entries to ground-truth intervals

## How to apply

After running the pipeline (`nextflow pipeline.nf --input files.csv` inside the antonipietryga/consensussv-ont-pipeline docker image), load the sample's consensus VCF from the vcfs output directory with pandas, skipping VCF header lines. Load the ground-truth deletion set (e.g. GIAB HG00514) and index its intervals by chromosome for fast overlap queries. For each consensus call flagged as a false positive by benchmarking (e.g. with Truvari bench), query overlapping ground-truth intervals and classify the call as merged (one prediction spanning two or more near-overlapping GT events), partial-overlap, or novel (no GT overlap). Validate the classifier against known examples: ground-truth deletions 933,966-934,190, 934,286-934,762 and 934,793-934,906 versus the single prediction 934,090-934,912 should be labelled 'merged'. Write one row per false positive (coordinates, class, list of overlapping GT regions) to fp_classification.csv.

## Related tools

- **ConsensuSV-ONT** (Produces the consensus SV VCFs (final CNN-filtered calls plus six individual caller VCFs) whose false positives this skill interprets) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the pipeline that generated the VCFs is executed) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline engine invoked inside the container (`nextflow pipeline.nf --input files.csv`) that emits the alignments/ and vcfs/ directories consumed by this skill) — github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
import pandas as pd; df = pd.read_csv('vcfs/HG00514/consensusSV.vcf', sep='\t', comment='#', header=None, names=['CHROM','POS','ID','REF','ALT','QUAL','FILTER','INFO'])
```

## Evaluation signals

- Classifier reproduces the Table 2 example: prediction 934,090-934,912 spanning GT deletions 933,966-934,190 / 934,286-934,762 / 934,793-934,906 is labelled 'merged'
- Every FP row in fp_classification.csv has non-empty coordinates, an assigned class from {merged, partial-overlap, novel}, and an interval list
- Novel calls have zero overlapping ground-truth intervals by construction; merged/partial classes must list ≥1 overlapping GT interval
- Coordinate counts match the deletion entries in the consensus VCF loaded from vcfs/<sample>/
- Consensus call set is smaller than the union of the six individual caller VCFs (CNN filtering removed low-quality variants)

## Limitations

- Near-overlapping but distinct ground-truth events can make a correct merged prediction appear as a false positive under Truvari bench, inflating FP counts
- The consensus output depends on CNN filtering quality; interpretation inherits any caller-level errors from the six upstream SV callers
- Benchmarking against a single ground truth (e.g. one GIAB sample) may not generalise; the paper notes a need for more detailed comparative analysis across tools and sequencing methods

## Evidence

- [readme] vcfs output folder: "vcfs - where you will find folders for each of the sample, containing all VCF files from the individual SV calling tools and consuSV-ONT"
- [abstract] CNN filtering context: "The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants"
- [discussion] performance basis for benchmarking: "ConsensuSV-ONT demonstrated a better performance than other tools used for long-read sequencing within the fully-created environment with an automated pipeline"
- [readme] pipeline invocation: "nextflow pipeline.nf --input files.csv"
