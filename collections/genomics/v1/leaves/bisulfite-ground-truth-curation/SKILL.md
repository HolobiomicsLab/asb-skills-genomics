---
name: bisulfite-ground-truth-curation
description: Use when benchmarking per-read or per-site 5mC methylation calls from
  Nanopore reads against short-read bisulfite ground truth to quantify caller accuracy
  via F1-score.
when_to_use_negative:
- Validating methylation calls against methylation microarray ground truth instead
  of short-read bisulfite — this skill's thresholds are defined for the EpiQC bisulfite
  benchmark
- Evaluating sites with intermediate 20–80% methylation — EpiQC reported poor concordance
  among short-read methods for these sites, so they must be excluded
- Calling methylation from signal data itself — this skill curates ground truth and
  scores calls; use the DeepMod2 methylation-calling workflow to generate output.per_read/output.per_site
  first
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0102
tools:
- name: DeepMod2
  role: Source of per-read (output.per_read) and per-site (output.per_site / output.per_site.aggregated)
    methylation predictions to be binarized and scored against ground truth
  repo: github.com/WGLab/DeepMod2
- name: modkit
  role: ONT tool used in the workflow to summarize modified-base (MM/ML tag) calls
    into per-site counts/bedMethyl for per-site evaluation
  repo: github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:d7c6299287f72bb5bea191db5b47e0697ae577144c7fd8dbbb43990969ae2322
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - modkit
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bisulfite-ground-truth-curation

## Summary

Curates extreme-methylation CpG ground truth from short-read bisulfite data (EpiQC, GEO GSE186383) and computes per-read and per-site precision/recall/F1 for Nanopore methylation callers such as DeepMod2 on HG002 R9.4.1 and R10.4.1 datasets.

## When to use

Use when benchmarking per-read or per-site 5mC methylation calls from Nanopore reads against short-read bisulfite ground truth to quantify caller accuracy via F1-score.

## When NOT to use

- Validating methylation calls against methylation microarray ground truth instead of short-read bisulfite — this skill's thresholds are defined for the EpiQC bisulfite benchmark
- Evaluating sites with intermediate 20–80% methylation — EpiQC reported poor concordance among short-read methods for these sites, so they must be excluded
- Calling methylation from signal data itself — this skill curates ground truth and scores calls; use the DeepMod2 methylation-calling workflow to generate output.per_read/output.per_site first

## Inputs

- DeepMod2 per-read methylation TSV output (output.per_read) with probability scores per CpG
- DeepMod2 per-site methylation outputs (output.per_site / output.per_site.aggregated)
- EpiQC short-read bisulfite ground truth from GEO accession GSE186383
- Genomic coordinates and strand for CpG matching (HG002 R9.4.1 and R10.4.1 flowcell datasets)

## Outputs

- Stratified F1 table (f1_scores_table.tsv) with flowcell, granularity, precision, recall, and F1 columns
- Per-read F1 for R9.4.1/R10.4.1 (expected 95.7–97.1% on R9.4.1 Ashkenazim trio)
- Per-site F1 with methylation-frequency labels (≥50% = methylated)

## How to apply

Download the EpiQC short-read bisulfite ground truth from GEO accession GSE186383 and stratify by granularity: restrict per-read ground truth to CpGs that are almost completely methylated (≥90% methylation frequency) or unmethylated (<10%) with ≥10X short-read coverage, and per-site ground truth to mostly methylated (≥80%) or mostly unmethylated (<20%) CpGs at ≥10X coverage for the Ashkenazim trio (HG002-4; ≥5X for NIH3T3). Exclude sites with 20–80% methylation entirely, because EpiQC reported poor concordance for intermediate sites both between ONT and short-read methods and among short-read methods themselves. Binarize DeepMod2 per-read predictions using the 0.5 probability-score threshold and per-site predictions using the ≥50% methylation-frequency cutoff, then match predicted CpGs to ground-truth CpGs by genomic coordinate and strand for each flowcell (R9.4.1 and R10.4.1) and granularity level. Compute precision, recall, and F1 per flowcell and granularity, and assemble results into a single table (f1_scores_table.tsv) with columns flowcell, granularity, precision, recall, F1. Validation targets: per-read F1 in the 95.7–97.1% range on R9.4.1 Ashkenazim trio, ~95% per-read and ~99% per-site F1 overall, and correlation r > 0.95 with short-read sequencing.

## Related tools

- **DeepMod2** (Source of per-read (output.per_read) and per-site (output.per_site / output.per_site.aggregated) methylation predictions to be binarized and scored against ground truth) — github.com/WGLab/DeepMod2
- **modkit** (ONT tool used in the workflow to summarize modified-base (MM/ML tag) calls into per-site counts/bedMethyl for per-site evaluation) — github.com/nanoporetech/modkit

## Examples

```
python PATH_TO_DEEPMOD2_REPOSITORY/deepmod2 detect --bam reads.bam --input INPUT_DIR --model MODEL --file_type FILE_TYPE --threads NUM_THREADS --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Per-read F1 on R9.4.1 Ashkenazim trio (HG002, HG003, HG004) falls in the 95.7–97.1% range
- Overall per-read F1 ≈ 95% and per-site F1 ≈ 99% on human cell lines against short-read ground truth
- Correlation with short-read sequencing exceeds r > 0.95
- f1_scores_table.tsv contains the expected columns: flowcell, granularity, precision, recall, F1
- No CpG with 20–80% methylation is present in the evaluated truth set; per-read truth sites have ≥10X short-read coverage

## Limitations

- Ground truth covers only extreme-methylation CpGs (≥90%/<10% per-read, ≥80%/<20% per-site); intermediate sites are excluded because of poor short-read concordance
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Filtering intermediate per-read predictions reduces coverage and statistical power for low-coverage samples

## Evidence

- [results] EpiQC ground truth accession: "we obtained ground truth from EpiQC study34, available from Gene Expression Omnibus (GEO) under the accession number GSE186383"
- [results] Per-read truth criteria: "the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X)"
- [results] Per-site truth criteria: "the ground truth only included sites that were mostly methylated (≥80% methylation) and mostly unmethylated (<20% methylation) with ample coverage (≥10X for HG002-4 and ≥5X for NIH3T3)"
- [results] Intermediate sites excluded: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
- [intro] Expected F1 and correlation: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] Per-read binarization threshold: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
