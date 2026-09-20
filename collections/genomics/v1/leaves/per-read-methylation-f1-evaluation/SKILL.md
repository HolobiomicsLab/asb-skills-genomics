---
name: per-read-methylation-f1-evaluation
description: Use when benchmarking per-read 5mC methylation calling accuracy of ONT
  callers (DeepMod2, Guppy, Dorado) on HG002/HG003/HG004 data against short-read bisulfite
  ground truth (GSE186383).
when_to_use_negative:
- Per-site (methylation frequency) accuracy is the target — per-site ground truth
  uses different filters (≥80%/<20% methylation, ≥50% frequency cutoff) and modkit
  aggregation
- The modification of interest is not 5mCpG — DeepMod2 only provides models for 5mCpG
  methylation
- Evaluating methylation at 20–80% intermediate methylation levels, which EpiQC showed
  have poor concordance even among short-read methods
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: DeepMod2
  role: Methylation caller under evaluation; produces per-read 5mC predictions and
    probability scores from ONT signal
  repo: https://github.com/WGLab/DeepMod2
- name: Guppy
  role: Comparative ONT caller with built-in 5mC calling; basecalling with --moves_out
    to retain move tables
- name: Dorado
  role: Comparative ONT caller; basecalling with --emit-moves to produce BAM with
    move tables
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to GRCh38 (-ax map-ont) while retaining move tables
    for signal alignment
- name: samtools
  role: Converts basecalled BAM to FASTQ with tags preserved (samtools fastq -T "*")
    before alignment
- name: modkit
  role: Manipulates modBAM MM/ML tags and generates bedMethyl summary counts
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:eb3ed9abee718e29935a76c438929e2cd0a00bc739d9556cbaf455f2742a2833
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# per-read-methylation-f1-evaluation

## Summary

Evaluates Nanopore methylation callers (DeepMod2, Guppy, Dorado) at the per-read level by binarizing 5mC probability calls at a 0.5 threshold and computing precision/recall/F1 against EpiQC short-read bisulfite ground truth, reproducing the ~95% per-read F1 reported for DeepMod2 on human cell lines.

## When to use

Use when benchmarking per-read 5mC methylation calling accuracy of ONT callers (DeepMod2, Guppy, Dorado) on HG002/HG003/HG004 data against short-read bisulfite ground truth (GSE186383).

## When NOT to use

- Per-site (methylation frequency) accuracy is the target — per-site ground truth uses different filters (≥80%/<20% methylation, ≥50% frequency cutoff) and modkit aggregation
- The modification of interest is not 5mCpG — DeepMod2 only provides models for 5mCpG methylation
- Evaluating methylation at 20–80% intermediate methylation levels, which EpiQC showed have poor concordance even among short-read methods

## Inputs

- POD5 or FAST5 signal files from ONT R9.4.1/R10.4.1 flowcells (Ashkenazim trio HG002, HG003, HG004)
- BAM file basecalled with Dorado --emit-moves or Guppy --moves_out and aligned to GRCh38 with move tables retained
- GRCh38 analysis-set reference FASTA (GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna.gz)
- EpiQC short-read bisulfite ground truth from GEO accession GSE186383

## Outputs

- Per-read methylation calls (output.per_read TSV / BED) with methylated/unmethylated labels at 0.5 probability threshold
- F1-score table (e.g. f1_scores_table.tsv or methylation_caller_comparison.tsv) with flowcell, caller, precision, recall, and F1 columns

## How to apply

Basecall POD5/FAST5 files with Dorado (--emit-moves) or Guppy (--moves_out) so the BAM carries move tables, then align to GRCh38 via samtools fastq -T "*" piped into minimap2 -ax map-ont to retain moves. Run DeepMod2 detect (e.g. bilstm_r10.4.1_5khz_v4.3 model, reference-anchored, --threads 8) and collect per-read outputs, binarizing per-read predictions at the 0.5 probability-score threshold. Download EpiQC short-read ground truth from GEO GSE186383 and restrict the per-read truth to CpG sites that are ≥90% or <10% methylated at ≥10X short-read coverage, excluding the poorly concordant 20–80% methylation range. Match predicted CpGs to ground-truth CpGs by genomic coordinate and strand, then compute precision, recall, and F1 per flowcell (R9.4.1 and R10.4.1). Validation: per-read F1 on R9.4.1 Ashkenazim trio should fall in the reported 95.7–97.1% range, and DeepMod2 F1 (97.5–98.2% on R10.4.1) should be comparable to Guppy's 97.9–98.5%.

## Related tools

- **DeepMod2** (Methylation caller under evaluation; produces per-read 5mC predictions and probability scores from ONT signal) — https://github.com/WGLab/DeepMod2
- **Guppy** (Comparative ONT caller with built-in 5mC calling; basecalling with --moves_out to retain move tables)
- **Dorado** (Comparative ONT caller; basecalling with --emit-moves to produce BAM with move tables) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to GRCh38 (-ax map-ont) while retaining move tables for signal alignment)
- **samtools** (Converts basecalled BAM to FASTQ with tags preserved (samtools fastq -T "*") before alignment)
- **modkit** (Manipulates modBAM MM/ML tags and generates bedMethyl summary counts) — https://github.com/nanoporetech/modkit

## Examples

```
python deepmod2 detect --bam aligned.bam --input pod5_dir --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --threads 8 --ref GRCh38.fa --output mod_calls
```

## Evaluation signals

- Per-read F1 on R9.4.1 Ashkenazim trio falls in the reported 95.7–97.1% range
- Per-read F1 on R10.4.1 (97.5–98.2% for DeepMod2) is comparable to Guppy's 97.9–98.5%
- Overall per-read F1 ~95% and per-site F1 ~99% with correlation r > 0.95 vs short-read sequencing
- Ground truth restricted to sites ≥90% or <10% methylated with ≥10X short-read coverage; intermediate 20–80% sites excluded
- Predictions binarized at 0.5 probability score; per-read labels matched to ground truth by coordinate and strand

## Limitations

- Ground truth excludes 20–80% methylation sites due to poor concordance among methods, so F1 does not reflect performance at intermediate methylation levels
- Move tables only approximately demarcate signal boundaries between consecutive bases, which can yield uninformative signal summary statistics
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels

## Evidence

- [intro] DeepMod2 achieves ~95% and ~99% F1-score for per-read and per-site evaluation on human cell lines: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] Per-read ground truth filter: "the benchmark ground truth only included sites that were almost completely methylated (≥90% methylation frequency) or unmethylated (<10% methylation frequency) with ample coverage (≥10X)"
- [results] Per-read binarization threshold: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [results] R9.4.1 per-read F1 range: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
- [readme] Basecalling with move tables requirement: [evidence span withheld — anchor status: external_doc]
