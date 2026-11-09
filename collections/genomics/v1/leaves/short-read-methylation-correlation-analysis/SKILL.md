---
name: short-read-methylation-correlation-analysis
description: 'Use when evaluating or comparing Nanopore 5mC methylation callers against
  short-read ground truth: you have per-read modBAM/TSV calls plus a short-read methylation
  reference (e.g. EpiQC, GSE186383) and need per-site F1 and frequency correlation.'
when_to_use_negative:
- Evaluating per-read (not per-site) performance — the per-read ground truth uses
  the stricter ≥90%/<10% methylation with ≥10X filter instead
- No short-read or microarray ground truth is available — correlation/F1 scoring cannot
  be computed
- Calling modifications other than 5mCpG — DeepMod2 provides models only for 5mCpG
  and no 5hmC was detected in GIAB cell lines HG001-HG007
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3301
tools:
- name: DeepMod2
  role: Generates per-read 5mC calls and per-site frequencies from ONT signal via
    BiLSTM/Transformer models that are benchmarked against short-read ground truth
  repo: github.com/WGLab/DeepMod2
- name: modkit
  role: Converts modBAM (MM/ML tags) into bedMethyl per-site methylation tables using
    best practices for downstream correlation scoring
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Basecaller used with --emit-moves to produce move tables required for signal
    alignment before methylation detection
  repo: https://cdn.oxfordnanoportal.com/software/analysis/
- name: Guppy
  role: Basecaller used with --bam_out --moves_out to retain move tables for DeepMod2;
    its built-in 5mC calls serve as the comparison baseline
- name: minimap2
  role: Aligns move-table-bearing basecalled reads to GRCh38 (-ax map-ont) for reference-anchored
    per-site methylation estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:66fb9b2579ec72080040641f4c07de78df3b0d74197ae0b92ec002a6b1adddd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# short-read-methylation-correlation-analysis

## Summary

Benchmark ONT per-site methylation calls (DeepMod2, Guppy, Dorado) against short-read sequencing ground truth by computing per-site F1 and correlation of methylation frequencies on the Ashkenazim trio (HG002, HG003, HG004) datasets, as done in the DeepMod2 paper.

## When to use

Use when evaluating or comparing Nanopore 5mC methylation callers against short-read ground truth: you have per-read modBAM/TSV calls plus a short-read methylation reference (e.g. EpiQC, GSE186383) and need per-site F1 and frequency correlation.

## When NOT to use

- Evaluating per-read (not per-site) performance — the per-read ground truth uses the stricter ≥90%/<10% methylation with ≥10X filter instead
- No short-read or microarray ground truth is available — correlation/F1 scoring cannot be computed
- Calling modifications other than 5mCpG — DeepMod2 provides models only for 5mCpG and no 5hmC was detected in GIAB cell lines HG001-HG007

## Inputs

- POD5 or FAST5 signal files from ONT R9.4.1/R10.4.1 flowcells (Ashkenazim trio HG002/HG003/HG004)
- BAM with move tables from Dorado (--emit-moves) or Guppy (--bam_out --moves_out)
- GRCh38 analysis-set reference FASTA (GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set)
- Short-read methylation ground truth (EpiQC study, GEO GSE186383) as methylation frequency per CpG

## Outputs

- per-site methylation frequency table per caller (bedMethyl or TSV)
- per-site F1-score for each caller (DeepMod2 vs Guppy vs Dorado)
- Pearson correlation of per-site methylation frequency vs short-read methylation
- methylation_caller_comparison.tsv summarizing per-read F1, per-site F1, and short-read correlation

## How to apply

Basecall POD5/FAST5 files with Dorado (--emit-moves) or Guppy (--moves_out), align to GRCh38 with minimap2 -ax map-ont retaining move tables (samtools fastq -T '*' pipe), then run DeepMod2 detect in reference-anchored mode. Convert per-read probability calls (threshold 0.5) into per-site methylation frequencies (e.g. via modkit pileup or DeepMod2 post-processing that combines per-read predictions). Restrict the ground truth to sites that are ≥80% or <20% methylated with ≥10X short-read coverage (≥10X for HG002-4, ≥5X for NIH3T3), and label CpG sites with ≥50% Nanopore methylation frequency as methylated; remove Nanopore sites below 10X coverage before scoring. Compute per-site F1 and the Pearson correlation between caller methylation frequency and short-read methylation frequency. Rationale: the paper shows per-site F1 ~99% and correlation r > 0.95 with short-reads, and excludes 20–80% methylation sites because EpiQC found poor concordance between ONT and short-read methods there.

## Related tools

- **DeepMod2** (Generates per-read 5mC calls and per-site frequencies from ONT signal via BiLSTM/Transformer models that are benchmarked against short-read ground truth) — github.com/WGLab/DeepMod2
- **modkit** (Converts modBAM (MM/ML tags) into bedMethyl per-site methylation tables using best practices for downstream correlation scoring) — https://github.com/nanoporetech/modkit
- **Dorado** (Basecaller used with --emit-moves to produce move tables required for signal alignment before methylation detection) — https://cdn.oxfordnanoportal.com/software/analysis/
- **Guppy** (Basecaller used with --bam_out --moves_out to retain move tables for DeepMod2; its built-in 5mC calls serve as the comparison baseline)
- **minimap2** (Aligns move-table-bearing basecalled reads to GRCh38 (-ax map-ont) for reference-anchored per-site methylation estimation)

## Examples

```
python deepmod2 detect --bam reads.bam --input INPUT_DIR --model bilstm_r10.4.1_5khz_v4.3 --file_type POD5 --threads 8 --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- DeepMod2 per-read F1 on R9.4.1 Ashkenazim trio should fall in the reported 95.7–97.1% range; R10.4.1 per-read F1 in the 97.5–98.2% range
- Per-site F1 should reach ~99% and correlation with short-reads should exceed r > 0.95 as reported in the paper
- Confirm the coverage filters were applied: <10X Nanopore sites removed for correlation; ground truth restricted to ≥10X (HG002-4) short-read coverage
- Verify binary per-site labelling used the ≥50% methylation-frequency threshold
- Check methylation_caller_comparison.tsv contains per-read F1, per-site F1, and correlation for all three callers

## Limitations

- Sites with 20–80% methylation are excluded from evaluation because EpiQC reported poor concordance between ONT and short-read methods in that range
- Move tables only approximate signal-to-basecall alignment, sometimes yielding uninformative signal summary statistics
- DeepMod2 shows a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Filtering intermediate per-read probability predictions (user-specified thresholds above 0.5) reduces coverage and statistical power, especially for low-coverage samples

## Evidence

- [results] Coverage filter before correlation scoring: "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [intro] Expected performance envelope: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] Per-site binarization threshold: "CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise"
- [results] Ground truth scope and concordance caveat: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method"
- [readme] Basecalling prerequisite for signal-based calls: [evidence span withheld — anchor status: external_doc]
