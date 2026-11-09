---
name: coordinate-based-methylation-label-handling
description: Use when converting per-read methylation probabilities into coordinate-anchored
  outputs (per-read TSV/BED, modBAM MM/ML tags, per-site frequencies) or when parsing
  label files in 'contig position strand label' format for CpG sites.
when_to_use_negative:
- Input is already a per-site bedMethyl or aggregated methylation frequency table
  rather than per-read signal-level predictions
- Only modBAM manipulation or summary statistics is needed and no signal files are
  available (modkit alone suffices)
- The target modification is not 5mCpG — DeepMod2 currently only provides models for
  5mCpG methylation
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: DeepMod2
  role: Produces per-read 5mC predictions at CpG coordinates, adds MM/ML tags to the
    BAM, and writes the per-read TSV output
  repo: github.com/WGLab/DeepMod2
- name: modkit
  role: Converts modBAM files with MM/ML tags to bedMethyl per-site summary tables
    using best practices
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Basecalls POD5/FAST5 with --emit-moves to create the move-table BAM aligned
    to the reference that anchors per-read methylation calls to coordinates
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:7608b55eff105ebd466c7368879ce23ab9763845783c25ef4c4129982a96cb47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - Dorado
  - PyTorch
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Coordinate-based methylation label handling

## Summary

Handles 5mC methylation calls keyed to genomic and read coordinates in the DeepMod2 framework: writing per-read TSV/BED records with chromosome/ref_position/read_position/strand/methylation_score columns, adding MM/ML tags to BAM, and aggregating per-read predictions into per-site methylation frequency.

## When to use

Use when converting per-read methylation probabilities into coordinate-anchored outputs (per-read TSV/BED, modBAM MM/ML tags, per-site frequencies) or when parsing label files in 'contig position strand label' format for CpG sites.

## When NOT to use

- Input is already a per-site bedMethyl or aggregated methylation frequency table rather than per-read signal-level predictions
- Only modBAM manipulation or summary statistics is needed and no signal files are available (modkit alone suffices)
- The target modification is not 5mCpG — DeepMod2 currently only provides models for 5mCpG methylation

## Inputs

- Aligned BAM with move tables from Dorado (--emit-moves) or Guppy (--bam_out --moves_out)
- POD5 or FAST5 raw signal files (R9.4.1 or R10.4.1 flowcells)
- Reference genome FASTA (e.g., GRCh38 with .fai index)
- Label files in 'contig position strand label' format (DeepMod2 sample training dataset release v0.3.1)

## Outputs

- Per-read methylation TSV/BED (read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg)
- Methylation-tagged BAM with MM/ML modification tags
- Per-site methylation frequency table (percentage of reads with methylated cytosine)

## How to apply

Run DeepMod2 on an aligned, move-table BAM plus POD5/FAST5 signal files with a reference FASTA so methylation calls are reference-anchored at CpG motif positions (--motif CG 0). For each CpG site on a read, the model yields a sigmoid probability; a threshold of 0.5 predicts methylated, and user-specified thresholds can exclude intermediate probability predictions. Write per-read predictions as a tab-separated file with columns read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg, and simultaneously add the predictions to the input BAM as MM/ML modification tags. In post-processing, combine all per-read predictions at each site to calculate the percentage of total reads with methylated cytosine; when building training labels from coordinate files, use coverage >=10 with >=90% methylation as positive and <10% as negative labels, matching the 'contig position strand label' format. The rationale is that reference anchoring via an aligned BAM is what allows signal- and read-level probabilities to be mapped to genomic CpG coordinates.

## Related tools

- **DeepMod2** (Produces per-read 5mC predictions at CpG coordinates, adds MM/ML tags to the BAM, and writes the per-read TSV output) — github.com/WGLab/DeepMod2
- **modkit** (Converts modBAM files with MM/ML tags to bedMethyl per-site summary tables using best practices) — https://github.com/nanoporetech/modkit
- **Dorado** (Basecalls POD5/FAST5 with --emit-moves to create the move-table BAM aligned to the reference that anchors per-read methylation calls to coordinates) — https://github.com/nanoporetech/dorado

## Examples

```
python deepmod2 detect --bam aligned.bam --input pod5_dir --model MODEL --file_type pod5 --threads 8 --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- The per-read output file exists and contains all eight columns (read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg)
- All methylation_score values fall in [0,1] (sigmoid probabilities)
- The output BAM carries MM/ML modification tags added by DeepMod2
- Applying the 0.5 threshold to probability scores yields binary methylated/unmethylated calls consistent with per-site frequencies computed across reads
- Label files parse cleanly in 'contig position strand label' format with coverage >=10 and labels at >=90% / <10% methylation for training sets

## Limitations

- Move tables only approximately demarcate signal boundaries between consecutive bases, which can yield uninformative signal summary statistics
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Filtering intermediate per-read predictions with stricter probability thresholds reduces coverage and statistical power, especially for low-coverage samples
- Only 5mCpG models exist; no 5hmC was detected in GIAB cell lines HG001-HG007, so other modifications cannot be handled

## Evidence

- [results] Per-read prediction output with threshold: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [results] BAM tags and TSV output: "DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output"
- [results] Per-read model input tied to CpG sites: "A feature matrix is fed to a BiLSTM or Transformer model that makes per-read predictions for each CpG site on the read"
- [results] Post-processing aggregation to per-site frequency: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [results] Move table boundary caveat: "the move table does not always accurately demarcate boundaries between signals associated with consecutive bases"
- [readme] Dorado basecalling with move tables for signal alignment: [evidence span withheld — anchor status: external_doc]
