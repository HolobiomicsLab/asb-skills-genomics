---
name: low-coverage-site-filtering
description: Use when computing Pearson correlations or accuracy metrics over per-site
  methylation frequencies from DeepMod2/Dorado calls (BED files), where low-coverage
  CpGs would otherwise distort agreement between RRMS, whole-genome ONT, and short-read
  datasets.
when_to_use_negative:
- The sample is inherently low coverage — the article notes filtering intermediate
  predictions and sites 'can lead to lower statistical power in downstream analyses,
  especially for low coverage samples'
- Per-read (rather than per-site) F1 evaluation is the goal — per-read ground truth
  uses ≥90%/<10% methylation labels on the read level, not a site-coverage filter
- Sites in the 20–80% intermediate methylation range are the target of interest, since
  EpiQC reported poor concordance there and this skill excludes them from evaluation
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: bedtools
  role: Intersects RRMS and whole-genome per-site BED outputs to keep only CpG sites
    present in both before the coverage filter
- name: modkit
  role: Converts DeepMod2 modBAM output to bedMethyl per-site counts used to apply
    the 10X coverage threshold
  repo: https://github.com/nanoporetech/modkit
- name: DeepMod2
  role: Produces the reference-anchored per-site methylation frequency BED files that
    are filtered and intersected
  repo: https://github.com/WGLab/DeepMod2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:4f989b97769300e065cbcd119eeb2ba45bc888c7061e3e71ae31424a5d5d772e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - DeepMod2
  - bedtools
  - minimap2
  - modkit
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# low-coverage-site-filtering

## Summary

Filters CpG sites with insufficient Oxford Nanopore coverage (<10X) and low-coverage/inconclusive ground-truth sites before comparing per-site methylation frequencies between RRMS and whole-genome Nanopore sequencing of HG002 or against short-read ground truth.

## When to use

Use when computing Pearson correlations or accuracy metrics over per-site methylation frequencies from DeepMod2/Dorado calls (BED files), where low-coverage CpGs would otherwise distort agreement between RRMS, whole-genome ONT, and short-read datasets.

## When NOT to use

- The sample is inherently low coverage — the article notes filtering intermediate predictions and sites 'can lead to lower statistical power in downstream analyses, especially for low coverage samples'
- Per-read (rather than per-site) F1 evaluation is the goal — per-read ground truth uses ≥90%/<10% methylation labels on the read level, not a site-coverage filter
- Sites in the 20–80% intermediate methylation range are the target of interest, since EpiQC reported poor concordance there and this skill excludes them from evaluation

## Inputs

- Per-site methylation frequency BED files (DeepMod2 output.per_site / output.per_site.aggregated) from RRMS and whole-genome Nanopore runs
- Short-read bisulfite or microarray ground-truth methylation site tables (e.g., EpiQC GSE186383)
- Reference-anchored modBAM/bedMethyl coverage counts (via modkit)

## Outputs

- Filtered set of overlapping CpG sites with Nanopore coverage ≥10X
- Paired per-site methylation percentages plus correlation value (e.g., methylation_correlation_scatter.tsv)
- Ground-truth evaluation set restricted to ≥80%/<20% methylation with ≥10X (HG002–4) or ≥5X (NIH3T3) coverage

## How to apply

After DeepMod2 detect produces output.per_site / output.per_site.aggregated BED files for each dataset, restrict analysis to CpG sites present in both the RRMS and whole-genome per-site outputs using bedtools intersect, and drop sites with Nanopore coverage below 10X. When benchmarking against short-read or array ground truth, also apply the article's ground-truth coverage cutoffs (≥10X for Ashkenazim trio HG002–HG004, ≥5X for NIH3T3) and retain only clearly methylated (≥80% or ≥90% frequency) or unmethylated (<20% or <10% frequency) sites, excluding the intermediate 20–80% range where EpiQC reported poor concordance. The rationale is that low-coverage sites carry high sampling noise in methylation frequency, and the article reports the headline r = 0.96 RRMS-vs-whole-genome correlation only after this filtering. The filtered, paired per-site values can then be written to a file such as methylation_correlation_scatter.tsv for correlation computation.

## Related tools

- **bedtools** (Intersects RRMS and whole-genome per-site BED outputs to keep only CpG sites present in both before the coverage filter)
- **modkit** (Converts DeepMod2 modBAM output to bedMethyl per-site counts used to apply the 10X coverage threshold) — https://github.com/nanoporetech/modkit
- **DeepMod2** (Produces the reference-anchored per-site methylation frequency BED files that are filtered and intersected) — https://github.com/WGLab/DeepMod2

## Examples

```
bedtools intersect -a rrms.per_site.bed -b wg.per_site.bed | awk '$5>=10 && $11>=10' > overlapping_cpgs_10x.bed
```

## Evaluation signals

- Every retained CpG site has Nanopore coverage ≥10X in both compared datasets
- The retained site set contains only CpGs present in both the RRMS and whole-genome per-site outputs (bedtools intersect verified)
- The Pearson r computed over the filtered overlapping CpGs closely matches the reported r = 0.96 (range 95.78–96.45%) between RRMS and whole-genome Nanopore sequencing
- Ground-truth benchmark sets contain only sites with ≥80% or <20% methylation frequency and adequate coverage (≥10X HG002–4 / ≥5X NIH3T3)

## Limitations

- Filtering out low-coverage and intermediate-probability sites reduces coverage and statistical power, 'especially for low coverage samples'
- Sites with 20–80% methylation had poor concordance between ONT and short-read methods, so filtering cannot rescue intermediate-methylation site agreement
- Move tables only approximate signal-to-basecall alignment, which can contribute to noisy per-site frequencies at some CpGs

## Evidence

- [results] after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively): "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [results] the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X): "the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X)"
- [results] EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
- [other] Use bedtools intersect to keep only CpG sites present in both the RRMS and whole-genome per-site outputs, dropping sites with Nanopore coverage below 10X.: [evidence span withheld — anchor status: not_found]
- [results] the resulting decrease in coverage for per-site predictions can lead to lower statistical power in downstream analyses, especially for low coverage samples: "the resulting decrease in coverage for per-site predictions can lead to lower statistical power in downstream analyses, especially for low coverage samples"
