---
name: per-position-methylation-accuracy-benchmarking
description: Use when ONT eventalign TSVs have been classified with trained mCaller
  models and you must quantify per-position detection accuracy against orthogonal
  methylated/unmethylated position labels.
when_to_use_negative:
- Classifying sequence contexts on which the model has not been trained — training
  biases can decrease accuracy at dissimilar contexts
- Comparing against R10.4 flowcell data — nanopolish eventalign does not currently
  support R10
- Validating non-motif methylation calls, which may be method-specific false positives
  reflecting different noise sources between nanopore current deviations and PacBio
  IPD ratios
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: mCaller
  role: Trains binary classifiers (NN, RF, NBC, LR) on current-deviation features
    and calls m6A per read at labeled positions or motifs
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes fastq/fast5 and produces scaled event-level TSVs via eventalign --scale-events
    for classification input
  repo: https://github.com/jts/nanopolish
- name: make_bed.py
  role: Summary script that aggregates per-read methylation predictions into a per-position
    bed file
  repo: https://github.com/al-mcintyre/mCaller
- name: mCaller_analysis_scripts
  role: Analysis scripts from the Nature Communications paper used for downstream
    aggregation and accuracy evaluation
  repo: https://github.com/al-mcintyre/mCaller_analysis_scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:353c6eafd27a8479758a83eff99228256fe9142410dc7bf0bc12b4212b0feed4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - bwa
  - mCaller
  - mCaller_analysis_scripts
  - make_bed.py
  - nanopolish
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# per-position-methylation-accuracy-benchmarking

## Summary

Benchmark m6A methylation classifiers on nanopore data by aggregating per-read calls to per-position methylation fractions and computing per-position accuracy against orthogonal labels (e.g. PacBio motif calls), as done for NN, RF, NBC, and LR classifiers in mCaller on E. coli R9 data.

## When to use

Use when ONT eventalign TSVs have been classified with trained mCaller models and you must quantify per-position detection accuracy against orthogonal methylated/unmethylated position labels.

## When NOT to use

- Classifying sequence contexts on which the model has not been trained — training biases can decrease accuracy at dissimilar contexts
- Comparing against R10.4 flowcell data — nanopolish eventalign does not currently support R10
- Validating non-motif methylation calls, which may be method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios

## Inputs

- Labeled positions file (chromosome, position, strand, label) from orthogonal PacBio m6A motif calls
- Albacore-basecalled R9 fastq plus fast5 indexed with nanopolish
- Bam aligned to the MG1655 reference with bwa mem -x ont2d and sorted with samtools
- Scaled event-level TSV from nanopolish eventalign --scale-events
- Trained mCaller model files (.pkl, e.g. model_NN_6_m6A.pkl)

## Outputs

- Per-read methylation probability files (*.eventalign.diffs.6) per classifier
- Bed file of per-position methylation predictions from make_bed.py
- Per-position accuracy table comparing classifiers against PacBio labels

## How to apply

Build a labeled positions file (chromosome, position, strand, label) from orthogonal PacBio m6A calls at known motifs such as GATC and AACNNNNNNGTGC in E. coli MG1655 K12. Train each classifier with mCaller.py --train (default six-variable 11-mer context, skip_thresh 0) and apply the saved .pkl models to a second same-strain dataset's nanopolish eventalign TSV (--scale-events) using -m GATC -b A. Then run make_bed.py to summarize per-read predictions (*.eventalign.diffs.6) into a per-position methylation bed file, aggregate calls to per-position methylation fractions, and compute per-position accuracy against the PacBio labels for each classifier. Success is judged by reproducing the reported ranking — NN ~81.3% exceeding RF ~80.8% — within the 60–98% per-position accuracy range reported across motifs.

## Related tools

- **mCaller** (Trains binary classifiers (NN, RF, NBC, LR) on current-deviation features and calls m6A per read at labeled positions or motifs) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes fastq/fast5 and produces scaled event-level TSVs via eventalign --scale-events for classification input) — https://github.com/jts/nanopolish
- **make_bed.py** (Summary script that aggregates per-read methylation predictions into a per-position bed file) — https://github.com/al-mcintyre/mCaller
- **mCaller_analysis_scripts** (Analysis scripts from the Nature Communications paper used for downstream aggregation and accuracy evaluation) — https://github.com/al-mcintyre/mCaller_analysis_scripts

## Examples

```
mCaller.py --train -c NN -p positions.txt -r reference.fasta -e data.eventalign.tsv -f data.fastq -d model_NN_6_m6A.pkl
```

## Evaluation signals

- Neural network per-position accuracy ~81.3% slightly exceeds random forest ~80.8% on the second same-strain E. coli dataset
- Per-position accuracy for each classifier falls within the reported 60–98% range across motifs
- Trained model files are produced for all four classifiers (model_NN/RF/NBC/LR_6_m6A.pkl) with default six-variable context and skip_thresh 0
- make_bed.py output bed file covers the labeled GATC/AACNNNNNNGTGC positions with methylation fractions comparable to PacBio labels

## Limitations

- mCaller showed 60–98% per-position accuracy at most motifs but notably failed at one S. aureus motif, indicating incomplete motif coverage
- Training biases could decrease accuracy at contexts with lower similarity to the training examples
- Both single-molecule methods can fail to distinguish between types of methylation (GATCGVNY motif in S. aureus)
- m6A is detected with lower per-read accuracy (~70%) than m5C (upwards of 80%) in nanopore data

## Evidence

- [intro] binary classifiers trained on current-deviation features: "train and test binary classifiers for the detection of m6A in nanopore data based on deviations between observed and expected currents"
- [discussion] accuracy range: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [methods] mCaller invocation: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] eventalign step: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
- [readme] mCaller README purpose: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
- [methods] make_bed summary step: "run summary script to generate a bed file of per-position methylation predictions"
