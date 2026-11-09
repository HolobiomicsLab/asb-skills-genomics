---
name: methylation-accuracy-benchmarking
description: Use when evaluating a methylation caller on PacBio HiFi reads with known
  ground-truth methylation states (PCR-treated unmethylated vs. M.SssI-methylated
  controls), e.g., to reproduce ccsmeth vs. HK model read-level accuracies or tune
  an ambiguous-call filter.
when_to_use_negative:
- The dataset lacks ground-truth methylation states (e.g., native human samples without
  paired PCR-treated and M.SssI-methylated control DNA), so read-level accuracy cannot
  be computed.
- The goal is haplotype-aware methylation phasing or allele-specific methylation detection
  rather than caller accuracy benchmarking — use the ccsmethphase pipeline instead.
- Detecting non-CpG 5mCs, hemimethylated CpGs, or other modifications such as 6mA
  — the article states these methods are incapable of detecting hemimethylated CpGs
  and cannot be applied to non-CpG 5mC or 6mA.
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: ccsmeth
  role: Deep-learning model producing read-level 5mCpG probability calls on PacBio
    HiFi reads; its accuracy (before and after ∆p filtering) is the quantity benchmarked
  repo: https://github.com/PengNi/ccsmeth
- name: HK model
  role: CNN-based holistic kinetic model trained on the same 50% training split and
    used as the read-level accuracy comparison baseline
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:25ac375857d46cf8430de6d20f8bf0b0c07d80e09eb385f167206af77dfff36f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - HK model
  - ccsmeth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# methylation-accuracy-benchmarking

## Summary

Benchmark read-level DNA 5mCpG detection accuracy of PacBio CCS deep-learning callers (e.g., ccsmeth vs. the HK model) on M.SssI-methylated/PCR-treated control read pairs, and quantify the accuracy gain from discarding ambiguous calls at a ∆p = 0.33 confidence threshold.

## When to use

Use when evaluating a methylation caller on PacBio HiFi reads with known ground-truth methylation states (PCR-treated unmethylated vs. M.SssI-methylated controls), e.g., to reproduce ccsmeth vs. HK model read-level accuracies or tune an ambiguous-call filter.

## When NOT to use

- The dataset lacks ground-truth methylation states (e.g., native human samples without paired PCR-treated and M.SssI-methylated control DNA), so read-level accuracy cannot be computed.
- The goal is haplotype-aware methylation phasing or allele-specific methylation detection rather than caller accuracy benchmarking — use the ccsmethphase pipeline instead.
- Detecting non-CpG 5mCs, hemimethylated CpGs, or other modifications such as 6mA — the article states these methods are incapable of detecting hemimethylated CpGs and cannot be applied to non-CpG 5mC or 6mA.

## Inputs

- PacBio CCS subreads.bam/hifi.bam from M.SssI-methyltransferase-treated and PCR-treated control DNA (M01&W01, M02&W02, M03&W03 pairs)
- Reference genome(s) for read alignment
- input_sheet.tsv with Group_ID, Sample_ID, Type, Path columns
- ccsmeth read-level per-CpG 5mCpG probability calls (Pr values) from the ccsmeth deep-learning models

## Outputs

- ccsmeth_read_level_benchmark.csv with read-level accuracy and AUC for ccsmeth vs. HK model on each dataset pair
- accuracy_metrics.tsv with pre- and post-filter accuracy and percentage of calls discarded by the ∆p = 0.33 filter
- Filtered set of high-confidence methylated/unmethylated per-read per-CpG calls

## How to apply

Download the controlled M.SssI-treated versus PCR-treated PacBio CCS dataset pairs (M01&W01, M02&W02, M03&W03) with aligned subreads.bam/hifi.bam and reference genomes organized in an input_sheet.tsv (Group_ID, Sample_ID, Type, Path). Split each pair 50/50 into training and test reads at the read level, train ccsmeth and the CNN-based HK model on the same training split, then evaluate read-level 5mCpG accuracy and AUC on the held-out test reads — ccsmeth should reach 0.9232, 0.8788, and 0.8765 on the three pairs, exceeding HK by 5.4%, 4.4%, and 3.7%. For the ambiguity-filter experiment, compute the confidence margin ∆p = |Pr − (1 − Pr)| for every per-read per-CpG probability, discard calls with ∆p < 0.33, and recompute accuracy; validation requires a 3.5-4.2 percentage-point accuracy gain while discarding 8.9-12.8% of calls. Write benchmark metrics to ccsmeth_read_level_benchmark.csv and filtered metrics to accuracy_metrics.tsv.

## Related tools

- **ccsmeth** (Deep-learning model producing read-level 5mCpG probability calls on PacBio HiFi reads; its accuracy (before and after ∆p filtering) is the quantity benchmarked) — https://github.com/PengNi/ccsmeth
- **HK model** (CNN-based holistic kinetic model trained on the same 50% training split and used as the read-level accuracy comparison baseline)

## Examples

```
delta_p = abs(pr - (1 - pr)); kept = [c for c in calls if delta_p(c) >= 0.33]; acc_before, acc_after, pct_discarded
```

## Evaluation signals

- Reproduced read-level accuracies match 0.9232, 0.8788, and 0.8765 on M01&W01, M02&W02, and M03&W03 within a small tolerance.
- ccsmeth's AUC exceeds the HK model's by approximately 5.4%, 4.4%, and 3.7% on the three dataset pairs.
- ∆p ≥ 0.33 filtering improves read-level accuracy by 3.5-4.2 percentage points while discarding 8.9-12.8% of calls.
- Filtered accuracy strictly exceeds unfiltered accuracy on the same held-out test split, confirming only ambiguous calls (small |Pr − (1 − Pr)|) were removed.
- On long (≥10 Kb) CCS reads, the unfiltered caller is consistent with the reported ~0.90 accuracy and ~0.97 AUC at single-molecule resolution.

## Limitations

- Filtering with ∆p = 0.33 trades accuracy for coverage: 8.9-12.8% of per-read per-CpG calls are discarded as ambiguous.
- These methods only consider symmetric methylation and are incapable of detecting hemimethylated CpGs.
- Applicability to non-CpG 5mC or other DNA modifications (e.g., 6mA) has not been verified; more ground-truth datasets and redesigned models are needed.
- The HK model baseline requires relatively high CCS subread depth (at least 20x passed subreads per CCS), limiting insert size and read length.

## Evidence

- [results] ccsmeth achieves read-level accuracies of 0.9232, 0.8788, and 0.8765 on M01&W01, M02&W02, and M03&W03, respectively: [evidence span withheld — anchor status: not_found]
- [results] ∆p filtering improves accuracy by 3.5-4.2% at 8.9-12.8% discarded: [evidence span withheld — anchor status: not_found]
- [abstract] ccsmeth 0.90 accuracy and 0.97 AUC on long CCS reads: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [readme] Delta probability call threshold explanation: [evidence span withheld — anchor status: external_doc]
- [discussion] ccsmeth outperforms existing CNN-based methods: "ccsmeth outperforms two existing CNN-based methods on both read-level and site-level 5mC prediction."
