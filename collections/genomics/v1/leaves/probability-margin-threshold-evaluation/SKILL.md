---
name: probability-margin-threshold-evaluation
description: Use when you have per-read per-CpG methylation probabilities (e.g., ccsmeth
  5mCpG calls on PacBio HiFi reads) with ground truth, and need to trade call coverage
  for accuracy via a confidence threshold.
when_to_use_negative:
- The modification is not CpG-context 5mC (e.g., non-CpG 5mC, 6mA, or hemimethylated
  CpGs) — ccsmeth models cannot detect these, so probability calls would not exist
  to filter
- No ground truth (methylated/unmethylated control DNA) is available, since the skill
  requires computing accuracy before vs. after filtering
- The downstream step is site-level aggregation at very low coverage, where discarding
  8.9–12.8% of reads may further degrade per-site estimates
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: ccsmeth
  role: Produces the per-read per-CpG 5mCpG probability calls whose ∆p = |Pr − P'r|
    margin is thresholded at 0.33
  repo: github.com/PengNi/ccsmeth
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:1a8438d39a7a09d7cd873ac386f1b083e22930a161ae5e9be7cfebe9966b6ae7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ccsmeth
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# probability-margin-threshold-evaluation

## Summary

Filters ambiguous per-read methylation calls by a probability-margin threshold ∆p = |Pr − P'r| (P'r = 1 − Pr), quantifying the accuracy gain against the fraction of calls discarded. Applied to ccsmeth read-level 5mCpG calls, ∆p ≥ 0.33 lifts read-level accuracy by 3.5–4.2 percentage points at the cost of dropping 8.9–12.8% of calls.

## When to use

Use when you have per-read per-CpG methylation probabilities (e.g., ccsmeth 5mCpG calls on PacBio HiFi reads) with ground truth, and need to trade call coverage for accuracy via a confidence threshold.

## When NOT to use

- The modification is not CpG-context 5mC (e.g., non-CpG 5mC, 6mA, or hemimethylated CpGs) — ccsmeth models cannot detect these, so probability calls would not exist to filter
- No ground truth (methylated/unmethylated control DNA) is available, since the skill requires computing accuracy before vs. after filtering
- The downstream step is site-level aggregation at very low coverage, where discarding 8.9–12.8% of reads may further degrade per-site estimates

## Inputs

- ccsmeth read-level per-read per-CpG 5mCpG probability calls (Pr values) from PacBio HiFi reads
- Ground-truth methylation states from PCR-treated (unmethylated) and M.SssI-methylated control DNA
- Threshold value ∆p = 0.33

## Outputs

- Filtered set of high-confidence methylated/unmethylated read-level calls
- Read-level accuracy before and after filtering
- Percentage of discarded calls (8.9–12.8% expected)
- accuracy_metrics.tsv

## How to apply

Start from ccsmeth read-level 5mCpG probability calls generated with the deep-learning models on PacBio HiFi reads whose methylation states are known from PCR-treated (unmethylated) and M.SssI-methylated control DNA. For every per-read per-CpG call compute the confidence margin ∆p = |Pr − P'r| where P'r = 1 − Pr; calls near Pr = 0.5 have ∆p near 0 and are ambiguous. Discard calls with ∆p below 0.33, keeping only high-confidence methylated/unmethylated calls. Compute read-level accuracy against the ground truth both before and after filtering, and record the percentage of calls discarded. Applied correctly, filtered accuracy improves by 3.5–4.2 percentage points while 8.9–12.8% of calls are discarded; write all metrics (e.g., to accuracy_metrics.tsv).

## Related tools

- **ccsmeth** (Produces the per-read per-CpG 5mCpG probability calls whose ∆p = |Pr − P'r| margin is thresholded at 0.33) — github.com/PengNi/ccsmeth

## Examples

```
calls["delta_p"] = (calls["Pr"] - (1 - calls["Pr"])).abs(); kept = calls[calls["delta_p"] >= 0.33]
```

## Evaluation signals

- Filtered read-level accuracy exceeds unfiltered accuracy by 3.5–4.2 percentage points
- Fraction of discarded calls falls within 8.9–12.8%
- Every retained call has ∆p = |Pr − (1 − Pr)| ≥ 0.33
- accuracy_metrics.tsv contains both pre-filter and post-filter accuracy plus discard rate
- Unfiltered ccsmeth accuracy should be ~0.90 (AUC ~0.97) on ≥10 Kb CCS reads, matching the published baseline

## Limitations

- Thresholding trades coverage for accuracy: 8.9–12.8% of calls are discarded, which can matter for downstream site-level aggregation under low coverage
- The underlying ccsmeth models only consider symmetric methylation and cannot detect hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA
- Evaluation requires ground-truth control DNA (PCR-treated unmethylated and M.SssI-methylated), limiting applicability to samples without such references

## Evidence

- [full_text] ∆p = 0.33 discard accuracy gain: [evidence span withheld — anchor status: not_found]
- [abstract] ccsmeth baseline accuracy: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [abstract] training data: "We sequence polymerase-chain-reaction treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth"
- [readme] delta probability threshold precedent: [evidence span withheld — anchor status: not_found]
