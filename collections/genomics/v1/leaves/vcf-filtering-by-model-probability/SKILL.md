---
name: vcf-filtering-by-model-probability
description: Use when a dysgu-generated SV VCF (or a merged multi-sample dysgu VCF
  combining paired-end, PacBio and ONT calls) needs quality filtering, especially
  before benchmarking or when combining callsets from different runs.
when_to_use_negative:
- Input VCFs come from callers other than dysgu, which do not emit the gradient-boosting
  model probability used for stratification
- Merging has not yet been performed — the README states filtering is generally recommended
  after any merging, not before
- Somatic/cohort analysis where a normal sample or panel of normals should be used
  with 'dysgu filter' rather than single-sample probability filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: dysgu
  role: Emits the model probability per SV event and provides the 'filter' command
    to remove lower-quality events from a VCF; 'merge' output is the typical filtering
    input
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Benchmarks the probability-filtered VCF against the GIAB HG002 truth set to
    verify the filtering threshold yields competitive precision/recall
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:e9e082804e6c9e17323b6231047cb4ed93fd5136f33d6717d2c7141917c3a54b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - svbench
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vcf-filtering-by-model-probability

## Summary

Filter structural variant calls in a VCF by dysgu's machine-learning model probability instead of manual pass/fail criteria, using calibrated thresholds such as max model probability ≥0.5 for PacBio and ≥0.35 for ONT callsets. This probability-based stratification underpins dysgu's merged PE+LR callsets benchmarked against the GIAB HG002 truth set.

## When to use

Use when a dysgu-generated SV VCF (or a merged multi-sample dysgu VCF combining paired-end, PacBio and ONT calls) needs quality filtering, especially before benchmarking or when combining callsets from different runs.

## When NOT to use

- Input VCFs come from callers other than dysgu, which do not emit the gradient-boosting model probability used for stratification
- Merging has not yet been performed — the README states filtering is generally recommended after any merging, not before
- Somatic/cohort analysis where a normal sample or panel of normals should be used with 'dysgu filter' rather than single-sample probability filtering

## Inputs

- dysgu SV callset in VCF format (single sample or merged output from 'dysgu merge')
- Merged Illumina PE + PacBio (or ONT) dysgu VCF awaiting quality filtering

## Outputs

- Filtered VCF retaining events at/above the maximum model probability threshold (≥0.5 PacBio, ≥0.35 ONT)
- Filtered callset suitable for svbench benchmarking against the GIAB HG002 truth set

## How to apply

Dysgu labels every putative SV event with a probability value produced by its gradient boosting classifier, so events can be stratified by probability rather than manually filtered. After running 'dysgu merge' on callsets from different runs (e.g. Illumina PE calls merged into PacBio or ONT long-read callsets), apply the probability threshold empirically chosen for the read-type combination: filter merged calls at a maximum model probability ≥0.5 for PacBio combinations, or ≥0.35 for ONT combinations (as used for Table 7). Filtering is generally recommended after any merging has been performed, or when analysing only a single sample, and the 'dysgu filter' command can remove lower-quality events from a single input VCF. The filtered VCF is then benchmarked against the GIAB HG002 truth set (Tier 1 and All-regions) with svbench to compute per-SV-type precision and recall.

## Related tools

- **dysgu** (Emits the model probability per SV event and provides the 'filter' command to remove lower-quality events from a VCF; 'merge' output is the typical filtering input) — https://github.com/kcleal/dysgu
- **svbench** (Benchmarks the probability-filtered VCF against the GIAB HG002 truth set to verify the filtering threshold yields competitive precision/recall) — https://github.com/kcleal/svbench

## Examples

```
dysgu filter input.vcf > output.vcf
```

## Evaluation signals

- Filtered PacBio merged callset retains events with maximum model probability ≥0.5; ONT merged callset retains events at ≥0.35
- Merged PacBio+PE and ONT+PE VCFs exist post-filtering and are valid VCF files
- svbench benchmark of the filtered merged callset shows the Table 7 pattern: added PE calls yield +1286 deletions and +894 insertions over 8× PacBio alone (All-regions), and +150 deletions / +33 insertions (Tier 1)
- Filtered merged low-coverage PE+LR callset is competitive in precision/recall with long-read-only calls at higher coverage
- Filtering is applied after merging, not before, matching the README recommendation

## Limitations

- Thresholds (≥0.5 PacBio, ≥0.35 ONT) were chosen empirically for the HG002 GIAB benchmark combinations; other read-type combinations may need retuning
- Stratifying by probability relies on a gradient boosting classifier trained on a manually labelled set, which induces an obvious bias in the training set
- The default divergence setting (0.02) can be too stringent for lower-accuracy long reads and will lower sensitivity; '--divergence auto' is recommended when read accuracy is uncertain
- Model probability is dysgu-specific — event labelling is 'a matter of opinion of the human observer' during training and does not transfer to other callers

## Evidence

- [results] Threshold selection for merged combinations: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [discussion] Probability-based stratification rationale: "Stratifying events by probability has several potential benefits over manually filtering."
- [results] Merge command and probability labelling: "Dysgu supports merging of SVs from different runs using a ‘merge’ command"
- [readme] Filter command usage from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Classifier labelling of events: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
