---
name: nanopore-signal-methylation-calling
description: Use when raw ONT signal data (POD5 or FAST5) from R9.4.1 or R10.4.1 flowcells
  must be basecalled, aligned, and analyzed for CpG 5mC methylation at per-read or
  per-site resolution, or benchmarked against short-read/microarray ground truth.
when_to_use_negative:
- Data is already basecalled modBAM/bedMethyl from Dorado modkit basecalling and no
  raw signal re-analysis is needed
- The target modification is not 5mCpG (e.g. 5hmC or 6mA), since DeepMod2 currently
  provides models only for 5mCpG
- Only R9.4-model-compatible third-party tools are acceptable for the analysis environment
  — DeepMod2 requires its own R9.4.1/R10.4.1 models
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: DeepMod2
  role: Deep-learning (BiLSTM/Transformer) methylation caller that consumes POD5/FAST5
    signals plus a move-table BAM and emits per-read and per-site 5mCpG calls
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecaller used with --emit-moves to generate a BAM with move tables that
    DeepMod2 needs for signal-to-base alignment
  repo: https://github.com/nanoporetech/dorado
- name: modkit
  role: Converts DeepMod2's modBAM output into bedMethyl per-site summary counts using
    best practices
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Aligns basecalled reads to the reference genome with -ax map-ont while preserving
    move-table tags for reference-anchored methylation calling
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:c1a824d66a42aa94f1d6952431082c06d008cc9bd7a1e00816418487f5b17116
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - Oxford Nanopore Technologies (ONT)
  - harmonicmeanp
  - minimap2
  - modkit
  - nanopolish
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-signal-methylation-calling

## Summary

Detect per-read and per-site 5mCpG DNA methylation from Oxford Nanopore ionic-current signals using DeepMod2's BiLSTM/Transformer models on POD5/FAST5 data basecalled with Guppy or Dorado. Produces methylation-tagged BAM (MM/ML tags) and per-read/per-site methylation outputs for R9.4.1 and R10.4.1 flowcells.

## When to use

Use when raw ONT signal data (POD5 or FAST5) from R9.4.1 or R10.4.1 flowcells must be basecalled, aligned, and analyzed for CpG 5mC methylation at per-read or per-site resolution, or benchmarked against short-read/microarray ground truth.

## When NOT to use

- Data is already basecalled modBAM/bedMethyl from Dorado modkit basecalling and no raw signal re-analysis is needed
- The target modification is not 5mCpG (e.g. 5hmC or 6mA), since DeepMod2 currently provides models only for 5mCpG
- Only R9.4-model-compatible third-party tools are acceptable for the analysis environment — DeepMod2 requires its own R9.4.1/R10.4.1 models

## Inputs

- POD5 or FAST5 raw signal files from ONT R9.4.1 or R10.4.1 flowcells
- Basecalled BAM with move tables (Dorado --emit-moves or Guppy --bam_out --moves_out)
- Reference genome FASTA (e.g. GRCh38.fa)
- DeepMod2 model specification (BiLSTM or Transformer checkpoint for the flowcell chemistry)

## Outputs

- Methylation-tagged BAM with MM and ML modification tags
- Per-read methylation TSV/BED output with per-CpG probability scores
- Per-site methylation frequency table (output.per_site / output.per_site.aggregated BED, convertible to bedMethyl via modkit)

## How to apply

First basecall FAST5/POD5 files with Dorado using --emit-moves (or Guppy with --bam_out --moves_out) to obtain a BAM containing move tables, which enable signal alignment with basecalls. Align basecalled reads to the reference genome (e.g. GRCh38) with minimap2 -ax map-ont while retaining move-table tags (samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y), since reference anchoring allows per-site methylation frequencies. Run DeepMod2's detect module supplying the aligned BAM, the signal directory, the appropriate BiLSTM model (e.g. bilstm_r10.4.1_5khz_v4.3), --file_type pod5, and --ref for reference-anchored calls; DeepMod2 extracts 19 signal summary statistics per base in a 21-bp window centered at each CpG, normalizes raw signal by median absolute deviation, and feeds the feature matrix to the model. A 0.5 probability threshold marks a CpG as methylated per read (user-specified thresholds can exclude intermediate predictions), and post-processing combines per-read predictions into a per-site percentage of reads with methylated cytosine. Use modkit or DeepMod2's post-processing to convert modBAM into bedMethyl per-site tables, and apply ≥10X Nanopore coverage filters before cross-method comparisons.

## Related tools

- **DeepMod2** (Deep-learning (BiLSTM/Transformer) methylation caller that consumes POD5/FAST5 signals plus a move-table BAM and emits per-read and per-site 5mCpG calls) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecaller used with --emit-moves to generate a BAM with move tables that DeepMod2 needs for signal-to-base alignment) — https://github.com/nanoporetech/dorado
- **modkit** (Converts DeepMod2's modBAM output into bedMethyl per-site summary counts using best practices) — https://github.com/nanoporetech/modkit
- **minimap2** (Aligns basecalled reads to the reference genome with -ax map-ont while preserving move-table tags for reference-anchored methylation calling)

## Examples

```
python deepmod2 detect --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --bam aligned.bam --input signal_dir --ref GRCh38.fa --threads 8 --output MOD_CALLS
```

## Evaluation signals

- Per-read F1-score against benchmark ground truth falls in the ~95% range reported for DeepMod2 on human cell lines (e.g. 95.7–97.1% on R9.4.1 Ashkenazim trio HG002/HG003/HG004)
- Per-site F1-score reaches ~99% against short-read ground truth, with correlation r > 0.95 with short-read sequencing
- Output BAM contains valid MM/ML modification tags and per-site frequencies are produced whenever an aligned BAM plus --ref are supplied
- RRMS vs whole-genome Nanopore per-site methylation frequencies over overlapping CpGs (≥10X coverage, bedtools intersect) yield Pearson r ≈ 0.96
- Pruned-model CPU runs should maintain per-read F1 near 0.95 while reducing inference from ~60 h to ~13 h

## Limitations

- Move tables provide only approximate signal-to-basecall alignment, and boundary demarcation errors can yield uninformative signal summary statistics
- DeepMod2 shows a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Only 5mCpG models exist; other modifications (e.g. 5hmC) are not detected (no 5hmC found in GIAB HG001–HG007)
- Filtering intermediate per-read probability predictions reduces coverage and statistical power, especially for low-coverage samples
- Read phasing may fail in long runs of homozygosity, limiting phased methylation analysis of imprinted regions

## Evidence

- [abstract] DeepMod2, a comprehensive deep-learning framework for methylation detection using ionic current signal from Nanopore sequencing: "Here we present DeepMod2, a comprehensive deep-learning framework for methylation detection using ionic current signal from Nanopore sequencing"
- [readme] POD5/FAST5 support and flowcell models: "DeepMod2 can call methylation from POD5 and FAST5 files basecalled with Guppy or Dorado and provides models for R10.4.1 and R9.4.1 flowcells."
- [readme] Basecalling with move tables: [evidence span withheld — anchor status: external_doc]
- [results] Post-processing to per-site frequencies: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [results] 0.5 methylation probability threshold: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used to exclude intermediate probability predictions"
