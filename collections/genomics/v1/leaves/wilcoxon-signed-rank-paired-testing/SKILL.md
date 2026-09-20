---
name: wilcoxon-signed-rank-paired-testing
description: Use when comparing two binners evaluated on the same set of samples with
  paired per-sample quality scores (e.g., CheckM/GUNC-classified high-quality bins
  per sample) and you need a significance P-value without assuming normality.
when_to_use_negative:
- The two sets of quality scores come from different, non-paired samples or projects
  (use an unpaired rank test instead)
- Inputs are already aggregated project-level counts rather than paired per-sample
  scores, since pairing information is required
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: SemiBin2
  role: binner under test whose per-sample high-quality bin counts form one side of
    each paired comparison
  repo: https://github.com/BigDataBiology/SemiBin/
- name: SemiBin1
  role: semi-supervised baseline binner whose per-sample scores are paired against
    SemiBin2's
  repo: https://github.com/BigDataBiology/SemiBin/
- name: VAMB
  role: baseline binner run on identical inputs whose per-sample scores are paired
    against SemiBin2's
- name: CheckM
  role: assesses completeness/contamination of bins to define the per-sample quality
    scores used in the test
- name: GUNC
  role: assesses chimerism of bins to classify high-quality bins feeding the paired
    scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:b5c18338cd0cef731d97a90c29a64869083b8d21c6c3fbca62cd52f4310f9847
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CheckM
  - GUNC
  - SemiBin1
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Wilcoxon signed-rank paired testing

## Summary

Non-parametric statistical test applied to paired per-sample metagenomic binning quality scores to determine whether one binner (SemiBin2) significantly outperforms another (SemiBin1 or VAMB) across the same samples.

## When to use

Use when comparing two binners evaluated on the same set of samples with paired per-sample quality scores (e.g., CheckM/GUNC-classified high-quality bins per sample) and you need a significance P-value without assuming normality.

## When NOT to use

- The two sets of quality scores come from different, non-paired samples or projects (use an unpaired rank test instead)
- Inputs are already aggregated project-level counts rather than paired per-sample scores, since pairing information is required

## Inputs

- paired per-sample binning quality scores (high-quality bin counts per sample) from two binners run on identical inputs
- CheckM completeness/contamination and GUNC chimerism assessments for every reconstructed bin
- sorted BAM files / bin outputs from SemiBin2, SemiBin1, and VAMB runs on the four projects

## Outputs

- Wilcoxon signed-rank P-values per binner pair per project
- rows appended to benchmark_results.csv with high-quality bin counts, percentage gains, and P-values

## How to apply

For each of the four real short-read projects (human gut, dog gut, ocean, soil with 82, 129, 109, and 101 samples), collect paired per-sample quality scores produced by the two binners being compared (e.g., SemiBin2 vs SemiBin1, SemiBin2 vs VAMB), where bins were assessed with CheckM (completeness/contamination) and GUNC (chimerism) to classify high-quality bins. Because the same samples are processed by both binners, the observations are paired, and the Wilcoxon signed-rank test is applied to the per-sample differences rather than an unpaired test. Record the resulting P-values alongside the counts of high-quality bins (SemiBin2 targets: 1678, 3776, 631, 254) and the percentage gains over SemiBin1 (8.3-21.5%) and VAMB (27.3-229.9%). A small P-value confirms that SemiBin2's improvement in high-quality bins is statistically significant, validating the reproduction against the Fig. 5 reported results; write all values to benchmark_results.csv.

## Related tools

- **SemiBin2** (binner under test whose per-sample high-quality bin counts form one side of each paired comparison) — https://github.com/BigDataBiology/SemiBin/
- **SemiBin1** (semi-supervised baseline binner whose per-sample scores are paired against SemiBin2's) — https://github.com/BigDataBiology/SemiBin/
- **VAMB** (baseline binner run on identical inputs whose per-sample scores are paired against SemiBin2's)
- **CheckM** (assesses completeness/contamination of bins to define the per-sample quality scores used in the test)
- **GUNC** (assesses chimerism of bins to classify high-quality bins feeding the paired scores)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(hq_semibin2, hq_baseline)
```

## Evaluation signals

- SemiBin2's high-quality bin gains fall within the reported 27.3-229.9% range vs VAMB and 8.3-21.5% range vs SemiBin1
- Wilcoxon P-values indicate the improvement of SemiBin2 over each baseline is statistically significant
- Reproduced Fig. 5 high-quality bin counts match the reported targets (1678, 3776, 631, 254) for the human gut, dog gut, ocean, and soil projects
- All counts, percentage gains, and P-values are present in benchmark_results.csv

## Limitations

- The test requires paired observations on identical inputs; binners must be run on the same assembled contigs and BAM files for a fair comparison
- Results depend on the high-quality bin definition from CheckM (completeness/contamination) and GUNC (chimerism); changing these criteria changes the paired scores

## Evidence

- [other] Apply the Wilcoxon signed-rank test to paired per-sample quality scores between binners and record the P-values.: [evidence span withheld — anchor status: not_found]
- [abstract] Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples."
- [other] Assess every reconstructed bin from the three binners with CheckM (completeness/contamination) and GUNC (chimerism) to classify high-quality bins per project.: [evidence span withheld — anchor status: not_found]
