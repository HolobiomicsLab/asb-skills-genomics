---
name: structural-variant-false-positive-classification
description: Use when a consensus SV caller (e.g. ConsensuSV-ONT on ONT reads) reports
  deletions that Truvari bench does not match to ground truth, and you need a per-FP
  mechanistic classification instead of a bare FP count.
when_to_use_negative:
- Input is a high-confidence consensus VCF that has already matched ground truth in
  Truvari bench (no FPs to classify).
- The ground-truth set is unavailable or lacks interval coordinates, so overlap-based
  classification cannot be performed.
- The goal is calling SVs from raw ONT fastq.gz data rather than post-hoc FP analysis
  of an existing ConsensuSV-ONT run.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ConsensuSV-ONT
  role: Source of the false-positive deletion calls being characterized against ground
    truth
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:78119cc10ece10e18b384ff15a7d881e6d3e308cdb2d1b36d27dab812d03735f
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structural-variant-false-positive-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Characterizes each false-positive deletion reported by ConsensuSV-ONT (e.g. on HG00514) against the ground-truth interval set, labeling each FP as merged (one prediction spanning multiple near-overlapping GT events), partial-overlap, or novel (no GT overlap).

## When to use

Use when a consensus SV caller (e.g. ConsensuSV-ONT on ONT reads) reports deletions that Truvari bench does not match to ground truth, and you need a per-FP mechanistic classification instead of a bare FP count.

## When NOT to use

- Input is a high-confidence consensus VCF that has already matched ground truth in Truvari bench (no FPs to classify).
- The ground-truth set is unavailable or lacks interval coordinates, so overlap-based classification cannot be performed.
- The goal is calling SVs from raw ONT fastq.gz data rather than post-hoc FP analysis of an existing ConsensuSV-ONT run.

## Inputs

- ConsensuSV-ONT deletion VCF for HG00514 from the vcfs output directory of a prior pipeline run
- Ground-truth deletion interval set for HG00514 (chromosome-indexed for overlap queries)

## Outputs

- fp_classification.csv with one row per FP: coordinates, assigned class (merged / partial-overlap / novel), list of overlapping GT regions

## How to apply

Load the ConsensuSV-ONT deletion VCF for the sample from the vcfs output directory with pandas, and load the ground-truth deletion set indexed by chromosome for interval-overlap queries. For each FP deletion, query the ground truth for overlapping intervals and assign a class: 'merged' when one prediction spans two or more near-overlapping GT events, 'partial-overlap' when intervals intersect incompletely, and 'novel' when no GT interval overlaps. Rationale: near-overlapping ground truth events cannot be collapsed by Truvari bench, so a merged prediction covering several GT deletions is counted as a false positive even though it reflects real signal. Validate the classifier by reproducing the published Table 2 example (GT deletions 933,966-934,190, 934,286-934,762, 934,793-934,906 versus prediction 934,090-934,912 must be labeled 'merged'), then write one row per FP with coordinates, assigned class, and overlapping GT regions to fp_classification.csv.

## Related tools

- **ConsensuSV-ONT** (Source of the false-positive deletion calls being characterized against ground truth) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
import pandas as pd; df = pd.read_csv('vcfs/HG00514/consensuSV-ONT.vcf', sep='\t', comment='#', header=None)
```

## Evaluation signals

- The classifier reproduces the Table 2 case: prediction 934,090-934,912 labeled 'merged' against GT events 933,966-934,190, 934,286-934,762, 934,793-934,906.
- Every FP row in fp_classification.csv carries exactly one of the classes merged / partial-overlap / novel.
- Each FP row lists at least one overlapping GT region unless its class is 'novel'.
- Row count of fp_classification.csv equals the FP deletion count from the ConsensuSV-ONT HG00514 VCF.

## Limitations

- Truvari bench cannot merge predictions across near-overlapping but separate GT events, inflating the FP count for merged events.
- The article provides no version numbers, parameters, or implementation details for ConsensuSV-ONT, so thresholds must be inferred or tuned empirically.
- FP classification depends on the completeness and interval accuracy of the ground-truth set; novel-class calls may still be true events absent from the benchmark.

## Evidence
