---
name: pearson-correlation-benchmark-computation
description: Use when you have per-site methylation frequencies from a Nanopore caller
  (e.g., output.per_site.aggregated from DeepMod2) plus matched short-read WGBS or
  methylation microarray ground truth, and need a genome-wide correlation benchmark
  per sample and flowcell.
when_to_use_negative:
- Evaluating per-read (not per-site) methylation accuracy — use per-read F1-score
  against binary ground truth (sites ≥90% or <10% methylated, ≥10X coverage) instead
  of frequency correlation
- Ground-truth sites with intermediate methylation (20–80%) should be excluded, as
  EpiQC reported poor concordance between ONT and short-read methods in that range
- Non-CpG methylation (e.g., 5hmC or non-CpG contexts) — DeepMod2 only provides models
  for 5mCpG methylation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepMod2
  role: Produces methylation-tagged BAMs and per-site outputs (output.per_site, output.per_site.aggregated)
    from POD5/FAST5 Nanopore signal files whose per-site CpG frequencies are correlated
    against ground truth
  repo: https://github.com/WGLab/DeepMod2
- name: modkit
  role: Used in the article's methods for working with modified-base data / bedMethyl
    summaries that feed per-site methylation frequency comparisons
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Basecalls POD5/FAST5 with move table output (e.g., --emit-moves) to prepare
    the BAM input required by DeepMod2 detect before correlation
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:7419fed663c77607fb77ba5537b3038d3ee122dff251f3c9cd8c07c7990b7982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - modkit
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pearson-correlation-benchmark-computation

## Summary

Compute genome-wide Pearson correlation coefficients between a Nanopore methylation caller's per-site CpG methylation frequencies (e.g., DeepMod2 output.per_site.aggregated) and short-read bisulfite sequencing or methylation microarray ground truth, benchmarking against the reported r > 0.95 for DeepMod2 on the Ashkenazim trio R9.4.1 and R10.4.1 datasets.

## When to use

Use when you have per-site methylation frequencies from a Nanopore caller (e.g., output.per_site.aggregated from DeepMod2) plus matched short-read WGBS or methylation microarray ground truth, and need a genome-wide correlation benchmark per sample and flowcell.

## When NOT to use

- Evaluating per-read (not per-site) methylation accuracy — use per-read F1-score against binary ground truth (sites ≥90% or <10% methylated, ≥10X coverage) instead of frequency correlation
- Ground-truth sites with intermediate methylation (20–80%) should be excluded, as EpiQC reported poor concordance between ONT and short-read methods in that range
- Non-CpG methylation (e.g., 5hmC or non-CpG contexts) — DeepMod2 only provides models for 5mCpG methylation

## Inputs

- DeepMod2 per-site aggregated methylation output (MOD_CALLS/output.per_site.aggregated) with per-CpG methylation frequencies (mod_percentage)
- Short-read bisulfite sequencing or methylation microarray ground-truth methylation levels (e.g., EpiQC study, GSE186383) for the Ashkenazim trio (HG002, HG003, HG004)
- Coverage information for Nanopore calls and ground truth to apply coverage filters
- bedtools intersect for intersecting shared CpG positions

## Outputs

- pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites
- Per-sample, per-flowcell Pearson correlation coefficients between DeepMod2 and ground-truth CpG methylation frequencies

## How to apply

Run DeepMod2 detect on POD5/FAST5 signal files with an aligned BAM and reference genome to produce output.per_site and output.per_site.aggregated files, then extract per-site CpG methylation frequencies (mod_percentage). Remove low-coverage CpG sites: <10X for Nanopore methylation callers and ≥10X for the Ashkenazim trio ground truth (≥5X for NIH3T3). Intersect DeepMod2 per-site frequencies with ground-truth methylation levels at shared CpG positions using bedtools intersect, then compute the Pearson correlation coefficient between DeepMod2 mod_percentage and ground-truth methylation frequency per sample per flowcell. Write all coefficients to pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites. Validation: each Pearson r must exceed 0.95, matching the DeepMod2 benchmark of 95–97.35% correlation against short-read sequencing on the Ashkenazim trio R9.4.1 and R10.4.1 datasets.

## Related tools

- **DeepMod2** (Produces methylation-tagged BAMs and per-site outputs (output.per_site, output.per_site.aggregated) from POD5/FAST5 Nanopore signal files whose per-site CpG frequencies are correlated against ground truth) — https://github.com/WGLab/DeepMod2
- **modkit** (Used in the article's methods for working with modified-base data / bedMethyl summaries that feed per-site methylation frequency comparisons) — https://github.com/nanoporetech/modkit
- **Dorado** (Basecalls POD5/FAST5 with move table output (e.g., --emit-moves) to prepare the BAM input required by DeepMod2 detect before correlation) — https://github.com/nanoporetech/dorado

## Examples

```
python PATH_TO_DEEPMOD2_REPOSITORY/deepmod2 detect --bam reads.bam --input INPUT_DIR --model bilstm_r9.4.1 --file_type POD5 --threads 16 --ref ref.fa --output MOD_CALLS; bedtools intersect -a output.per_site.aggregated -b ground_truth.bed | awk computes Pearson r per sample/flowcell into pearson_correlation_table.csv
```

## Evaluation signals

- Every computed Pearson r in pearson_correlation_table.csv exceeds 0.95, matching the published DeepMod2 benchmark (95–97.35% genome-wide CpG methylation-frequency correlation)
- Per-site and aggregated output files (output.per_site, output.per_site.aggregated) exist and contain mod_percentage per CpG
- Low-coverage CpGs are removed: <10X Nanopore sites and <10X Ashkenazim trio ground-truth sites (≥5X for NIH3T3) excluded before correlation
- n_cpg_sites column is populated and non-zero for each sample/flowcell row, reflecting the number of shared CpG positions retained after intersection
- Intermediate-methylation ground-truth sites (20–80%) excluded from the evaluation set

## Limitations

- Sites with 20–80% methylation had poor concordance between ONT and short-read methods (per EpiQC), so correlation is computed on near-binary methylation states only, inflating apparent agreement
- Move tables do not always accurately demarcate signal boundaries between consecutive bases, which can lead to uninformative signal summary statistics and affect per-site frequencies
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels

## Evidence

- [results] DeepMod2 BiLSTM achieves the highest genome-wide CpG methylation-frequency correlation among the tested tools on the Ashkenazim trio R9.4.1 and R10.4.1 datasets, at 95–97.35%.: [evidence span withheld — anchor status: not_found]
- [results] after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively): "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [intro] DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
- [readme] This will give you a per-read prediction text file MOD_CALLS/output.per_read, a per-site prediction file MOD_CALLS/output.per_site, a per-site prediction file with both strands aggregated MOD_CALLS/output.per_site.aggregated: [evidence span withheld — anchor status: external_doc]
