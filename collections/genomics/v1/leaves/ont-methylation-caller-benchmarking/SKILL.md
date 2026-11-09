---
name: ont-methylation-caller-benchmarking
description: Use when comparing methylation-caller accuracy (e.g., DeepMod2 vs Guppy/Dorado)
  on Ashkenazim trio (HG002/HG003/HG004) ONT datasets against short-read ground truth
  using per-read/per-site F1 and correlation.
when_to_use_negative:
- The goal is calling 5hmC, 6mA, or other modifications rather than benchmarking 5mCpG
  — DeepMod2 currently only provides models for 5mCpG methylation
- Evaluating methylation accuracy at sites with intermediate (20–80%) methylation
  — ground truth concordance there is poor even among short-read methods
- The task is de novo methylation calling on new samples, not accuracy evaluation
  — no short-read ground truth is needed then and this skill's filtering/scoring does
  not apply
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DeepMod2
  role: Benchmarked open-source methylation caller; detect module produces per-read
    5mC predictions and MM/ML-tagged BAM from move-table BAM plus POD5/FAST5 signal
  repo: https://github.com/WGLab/DeepMod2
- name: Guppy
  role: ONT baseline caller; basecall with --moves_out and run built-in 5mC calling
    pipeline for comparison
- name: Dorado
  role: ONT baseline caller; basecall with --emit-moves and run modified basecalling
    (5mCG) for comparison
  repo: https://github.com/nanoporetech/dorado
- name: modkit
  role: Converts modBAM (MM/ML tags) from each caller into bedMethyl per-site summary
    counts for per-site evaluation
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Aligns move-table-bearing basecalled reads to GRCh38 (map-ont) while preserving
    move tables for signal alignment
- name: samtools
  role: samtools fastq -T converts basecalled BAM to FASTQ retaining move-table tags
    for minimap2 piping
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:0d4c0d61d8a0b86fa9af034175daf35ed3d17494f1470ad1b574b7655aa00408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ont-methylation-caller-benchmarking

## Summary

Benchmark per-read and per-site 5mC methylation-calling accuracy of ONT tools (DeepMod2, Guppy, Dorado) on R9.4.1/R10.4.1 Nanopore data by scoring against filtered short-read WGBS ground truth with F1-scores and per-site methylation-frequency correlation.

## When to use

Use when comparing methylation-caller accuracy (e.g., DeepMod2 vs Guppy/Dorado) on Ashkenazim trio (HG002/HG003/HG004) ONT datasets against short-read ground truth using per-read/per-site F1 and correlation.

## When NOT to use

- The goal is calling 5hmC, 6mA, or other modifications rather than benchmarking 5mCpG — DeepMod2 currently only provides models for 5mCpG methylation
- Evaluating methylation accuracy at sites with intermediate (20–80%) methylation — ground truth concordance there is poor even among short-read methods
- The task is de novo methylation calling on new samples, not accuracy evaluation — no short-read ground truth is needed then and this skill's filtering/scoring does not apply

## Inputs

- ONT POD5/FAST5 signal files from R9.4.1 and R10.4.1 flowcells (Ashkenazim trio HG002, HG003, HG004)
- Basecalled BAM with move tables (Dorado --emit-moves or Guppy --bam_out --moves_out)
- GRCh38 analysis-set reference FASTA (GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna.gz)
- Short-read WGBS methylation ground truth (EpiQC study, GEO GSE186383)
- DeepMod2 model weights (e.g. bilstm_r10.4.1_5khz_v4.3)

## Outputs

- Methylation-tagged BAM files (MM/ML tags) and per-read TSV predictions from each caller
- bedMethyl per-site methylation tables (via modkit)
- Caller comparison table (methylation_caller_comparison.tsv) with per-read F1, per-site F1, and short-read correlation per caller per sample

## How to apply

Basecall POD5/FAST5 signal files with Dorado (--emit-moves, e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0) or Guppy (--bam_out --moves_out) so the BAM carries move tables, then align to GRCh38 (GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set) with samtools fastq -T piped into minimap2 -ax map-ont to retain move tables. Run each caller on the same datasets: DeepMod2's detect module (e.g. bilstm_r10.4.1_5khz_v4.3, reference-anchored, 0.5 probability threshold) plus the built-in 5mC pipelines of Guppy and Dorado, and aggregate per-site frequencies with modkit. Build ground truth from short-read methylation (EpiQC, GSE186383): per-read evaluation keeps sites ≥90% or <10% methylated with ≥10X coverage; per-site evaluation keeps sites ≥80% or <20% methylated with ≥10X (HG002-4) or ≥5X (NIH3T3) coverage. Label CpG sites with methylation frequency ≥50% as methylated, convert per-read probability calls at threshold 0.5, and compute per-read F1, per-site F1, and Pearson correlation of per-site methylation frequency with short reads after removing low-coverage (<10X Nanopore) sites. The rationale is that intermediate methylation (20–80%) is excluded because EpiQC showed poor concordance even among short-read methods, so benchmarking is only meaningful on near-homogeneous sites.

## Related tools

- **DeepMod2** (Benchmarked open-source methylation caller; detect module produces per-read 5mC predictions and MM/ML-tagged BAM from move-table BAM plus POD5/FAST5 signal) — https://github.com/WGLab/DeepMod2
- **Guppy** (ONT baseline caller; basecall with --moves_out and run built-in 5mC calling pipeline for comparison)
- **Dorado** (ONT baseline caller; basecall with --emit-moves and run modified basecalling (5mCG) for comparison) — https://github.com/nanoporetech/dorado
- **modkit** (Converts modBAM (MM/ML tags) from each caller into bedMethyl per-site summary counts for per-site evaluation) — https://github.com/nanoporetech/modkit
- **minimap2** (Aligns move-table-bearing basecalled reads to GRCh38 (map-ont) while preserving move tables for signal alignment)
- **samtools** (samtools fastq -T converts basecalled BAM to FASTQ retaining move-table tags for minimap2 piping)

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 pod5s/ --emit-moves > basecall.bam; samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y -t 8 | samtools view -o aligned.bam
```

## Evaluation signals

- DeepMod2 per-read F1 on R9.4.1 Ashkenazim trio falls in the reported 95.7–97.1% range
- On R10.4.1 trio data, DeepMod2 per-read F1 of 97.5–98.2% is comparable to Guppy's 97.9–98.5%
- Per-read F1 ≈95% and per-site F1 ≈99% on human cell lines, with correlation r > 0.95 with short-read sequencing
- Ground-truth filters verified: per-read ≥90%/<10% methylation at ≥10X, per-site ≥80%/<20% at ≥10X (HG002-4)/≥5X (NIH3T3) coverage
- Per-read calls use the 0.5 probability threshold and per-site labels use the ≥50% methylation-frequency cutoff

## Limitations

- Sites with 20–80% methylation are excluded from evaluation because EpiQC reported poor concordance, so benchmark scores do not cover intermediate methylation states
- Move tables only approximately demarcate signal boundaries between consecutive bases, which can yield uninformative signal summary statistics
- DeepMod2 shows a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Filtering intermediate per-read probability predictions reduces per-site coverage and statistical power, especially for low-coverage samples
- DeepMod2 only provides 5mCpG models due to lack of training data for other modifications (no 5hmC detected in GIAB HG001-HG007)

## Evidence

- [results] For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
- [results] the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X): "the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X)"
- [results] CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise: "CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise"
- [results] A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [results] EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
- [readme] Basecall your FAST5/POD5 files with Dorado (using `--emit-moves`) or Guppy (using `--bam_out --moves_out`) parameters to get a BAM file with move tables: [evidence span withheld — anchor status: external_doc]
- [results] the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics: "the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics"
