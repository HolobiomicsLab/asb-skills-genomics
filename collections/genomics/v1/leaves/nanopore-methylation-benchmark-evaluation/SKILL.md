---
name: nanopore-methylation-benchmark-evaluation
description: Use when evaluating methylation-calling accuracy of a Nanopore 5mC caller
  against short-read bisulfite ground truth at per-read and per-site granularity,
  needing stratified F1 scores with explicit methylation-frequency filters.
when_to_use_negative:
- The task is to generate methylation calls rather than evaluate them — DeepMod2 detection
  is a separate step; this skill assumes existing per-read/per-site outputs
- Evaluation requires sites with intermediate 20–80% methylation — EpiQC reported
  poor concordance there between ONT and short-read methods, so F1 on such sites is
  not a reliable signal
- Non-5mC modifications (e.g., 5hmC) are of interest — DeepMod2 models cover only
  5mCpG methylation and no 5hmC was detected in GIAB cell lines HG001-HG007
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: DeepMod2
  role: Source of per-read (0.5 probability threshold) and per-site (≥50% frequency)
    5mC methylation predictions being benchmarked
  repo: https://github.com/WGLab/DeepMod2
- name: modkit
  role: Ont/best-practice tool for summarizing modBAM MM/ML tags into bedMethyl per-site
    counts used in per-site comparison
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:10cf61ba568c52cd4d296a3b5a1a631735d50fb733af00e0b9024046c7bae085
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - DeepSignal
  - METEORE
  - Megalodon
  - modkit
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- nanopore-cpg-methylation-benchmark-evaluation
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nanopore-methylation-benchmark-evaluation

## Summary

Benchmark 5mC methylation calls from a Nanopore methylation caller (e.g., DeepMod2) against bisulfite short-read ground truth (EpiQC, GSE186383), computing per-read (~95% expected) and per-site (~99% expected) precision, recall, and F1-score on HG002-class human datasets across R9.4.1 and R10.4.1 flowcells.

## When to use

Use when evaluating methylation-calling accuracy of a Nanopore 5mC caller against short-read bisulfite ground truth at per-read and per-site granularity, needing stratified F1 scores with explicit methylation-frequency filters.

## When NOT to use

- The task is to generate methylation calls rather than evaluate them — DeepMod2 detection is a separate step; this skill assumes existing per-read/per-site outputs
- Evaluation requires sites with intermediate 20–80% methylation — EpiQC reported poor concordance there between ONT and short-read methods, so F1 on such sites is not a reliable signal
- Non-5mC modifications (e.g., 5hmC) are of interest — DeepMod2 models cover only 5mCpG methylation and no 5hmC was detected in GIAB cell lines HG001-HG007

## Inputs

- DeepMod2 per-read methylation output TSV (output.per_read) for HG002 R9.4.1 and R10.4.1 datasets
- DeepMod2 per-site outputs (output.per_site, output.per_site.aggregated)
- EpiQC short-read bisulfite ground truth from GEO accession GSE186383
- Reference genome FASTA for CpG coordinate/strand matching

## Outputs

- Stratified F1-score table (f1_scores_table.tsv) with columns flowcell, granularity, precision, recall, F1
- Per-read F1 / per-site F1 values benchmarked against bisulfite ground truth
- Correlation estimate between Nanopore and short-read methylation frequencies

## How to apply

Load the caller's per-read outputs (e.g., DeepMod2 output.per_read TSV) and per-site outputs (output.per_site / output.per_site.aggregated), and download the EpiQC short-read bisulfite ground truth from GEO accession GSE186383. Restrict per-read ground truth to CpGs with ≥90% or <10% methylation frequency at ≥10X short-read coverage, and per-site ground truth to CpGs with ≥80% or <20% methylation at ≥10X coverage (HG002-4 criteria), excluding 20–80% methylation sites because EpiQC reported poor concordance there among methods. Binarize per-read predictions using the 0.5 probability-score threshold and per-site predictions using the ≥50% methylation-frequency cutoff, then match predicted CpGs to ground-truth CpGs by genomic coordinate and strand for each flowcell (R9.4.1, R10.4.1) and granularity level. Compute precision, recall, and F1 per flowcell and granularity, assembling results into a table (e.g., f1_scores_table.tsv); correct application reproduces per-read F1 in the 95.7–97.1% range on R9.4.1 Ashkenazim trio samples and ~95% per-read / ~99% per-site F1 overall with correlation r > 0.95.

## Related tools

- **DeepMod2** (Source of per-read (0.5 probability threshold) and per-site (≥50% frequency) 5mC methylation predictions being benchmarked) — https://github.com/WGLab/DeepMod2
- **modkit** (Ont/best-practice tool for summarizing modBAM MM/ML tags into bedMethyl per-site counts used in per-site comparison) — https://github.com/nanoporetech/modkit

## Examples

```
python deepmod2 detect --bam reads.bam --input INPUT_DIR --model MODEL --file_type FILE_TYPE --threads NUM_THREADS --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Per-read F1 on R9.4.1 Ashkenazim trio (HG002, HG003, HG004) falls in the 95.7–97.1% range
- Overall F1 approximates ~95% per-read and ~99% per-site against short-read sequencing ground truth
- Correlation between Nanopore and short-read methylation frequencies exceeds r > 0.95
- Evaluation set excludes ground-truth CpGs in the 20–80% methylation band and includes only sites with ≥10X coverage
- f1_scores_table.tsv has exactly the columns flowcell, granularity, precision, recall, F1 for both R9.4.1 and R10.4.1 flowcells

## Limitations

- EpiQC reported that sites with 20–80% methylation have poor concordance not only between ONT and short-reads but also among different short-read methods, so they are excluded from evaluation
- DeepMod2 shows a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- DeepMod2 provides models only for 5mCpG methylation due to a lack of available datasets for other methylation types
- Filtering intermediate per-read predictions reduces coverage and can lower statistical power for downstream analyses, especially for low coverage samples

## Evidence

- [results] For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
- [intro] DeepMod2 achieves ~95% and ~99% F1-score for per-read and per-site evaluation: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] Per-read ground truth filtering criteria: ≥90% or <10% methylation frequency at ≥10X coverage: "the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X)"
- [results] Per-site threshold: CpG sites with methylation frequency ≥50% labelled methylated: "CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise"
- [results] Exclusion of 20–80% methylation sites due to poor concordance among methods: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
- [results] Per-read binarization threshold of 0.5 for the probability score: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [results] Ground truth obtained from the EpiQC study via GEO accession GSE186383: "we obtained ground truth from EpiQC study34, available from Gene Expression Omnibus (GEO) under the accession number GSE186383"
