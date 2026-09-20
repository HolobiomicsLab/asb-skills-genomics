---
name: precision-recall-auc-benchmark-comparison
description: Use when comparing base-modification callers on the same ONT reads/alignments,
  where methyltransferase motif sites define positives and an unmethylated strain's
  motif sites define negatives, and per-motif PR-AUC is the metric.
when_to_use_negative:
- Reads come from R10.4 flowcells — nanopolish states variant and methylation calling
  is accurate enough without signal-level analysis and eventalign support has no timeline
- The evaluation goal is m5C rather than m6A — the article notes nanopore detects
  m5C with higher per-read accuracy (~80%) but the models here are m6A-specific (r95_twobase_model_NN_6_m6A.pkl,
  -b A)
- No ground-truth unmethylated control is available — the comparison design depends
  on P. aeruginosa unmethylated motif sites as true negatives
tools:
- name: mCaller
  role: Per-read m6A methylation probability caller using deviations between measured
    and expected currents; one arm of the benchmark
  repo: https://github.com/al-mcintyre/mCaller
- name: Tombo
  role: Supplies the m6A alternative-model per-position scores; comparison arm of
    the benchmark
- name: nanopolish
  role: Indexing and eventalign (--scale-events, -n) to produce the event TSV shared
    by both callers
  repo: https://github.com/jts/nanopolish
- name: bwa mem
  role: Aligns ONT fastq reads to the reference with -x ont2d; alternative aligner
    graphmap also supported
- name: scikit-learn
  role: Computes precision-recall curves and AUC per motif for the comparison
- name: samtools
  role: Sorts and indexes the BAM produced by bwa mem
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:3f6bdf7e323e6660eba363d472393f70640f973848b4d79bd9a7eae1c61f620e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - Tombo
  - bwa mem
  - mCaller
  - nanopolish
  - samtools
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# precision-recall-auc-benchmark-comparison

## Summary

Benchmark per-position m6A detection methods (mCaller vs Tombo) by computing per-motif precision-recall AUC with scikit-learn on nanopore data, using unmethylated motif sites from P. aeruginosa as true negatives and validating against the published Fig 4 comparison (GATC 0.91 vs 0.83; AAGANNNNNCTC 0.71 vs 0.86; GAGNNNNNTCTT 0.90 vs 0.64).

## When to use

Use when comparing base-modification callers on the same ONT reads/alignments, where methyltransferase motif sites define positives and an unmethylated strain's motif sites define negatives, and per-motif PR-AUC is the metric.

## When NOT to use

- Reads come from R10.4 flowcells — nanopolish states variant and methylation calling is accurate enough without signal-level analysis and eventalign support has no timeline
- The evaluation goal is m5C rather than m6A — the article notes nanopore detects m5C with higher per-read accuracy (~80%) but the models here are m6A-specific (r95_twobase_model_NN_6_m6A.pkl, -b A)
- No ground-truth unmethylated control is available — the comparison design depends on P. aeruginosa unmethylated motif sites as true negatives

## Inputs

- ONT nanopore FASTQ reads with matching FAST5 files (R9.4/R9.5, basecalled with Albacore or another event-preserving basecaller)
- Reference genome FASTA (e.g. E. coli MG1655 K12, E. faecalis)
- Positions/motif file listing methylated motif sites (GATC, AAGANNNNNCTC, GAGNNNNNTCTT)
- nanopolish eventalign TSV with scaled event values
- Per-read methylation probability outputs from mCaller and per-position scores from Tombo

## Outputs

- pr_auc_comparison.csv with per-motif precision-recall AUC values for mCaller vs Tombo
- Bed file of per-position methylation predictions from make_bed.py
- Precision-recall curves per motif generated with scikit-learn

## How to apply

Prepare event-level features once and run both callers on them: index the ONT fastq/fast5 with nanopolish index, align with bwa mem (-x ont2d) plus samtools sort/index, then run nanopolish eventalign with --scale-events and -n to emit the TSV of event values scaled toward the model. Run mCaller.py with the R9.5 model r95_twobase_model_NN_6_m6A.pkl (-b A, default 6 variables = 11-mer context) and Tombo with its m6A alternative model on the same reads and BAM. Aggregate per-read calls into per-position summaries with make_bed.py, applying a minimum read depth (-d) and a mod_threshold of 0.5 (minimum fraction of reads labelled methylated). Compute precision-recall curves and AUC per motif (GATC, AAGANNNNNCTC, GAGNNNNNTCTT) with scikit-learn, treating P. aeruginosa unmethylated motif sites as true negatives, and write pr_auc_comparison.csv. Judge correctness by whether the per-motif AUCs reproduce the published Fig 4 values within a reasonable tolerance.

## Related tools

- **mCaller** (Per-read m6A methylation probability caller using deviations between measured and expected currents; one arm of the benchmark) — https://github.com/al-mcintyre/mCaller
- **Tombo** (Supplies the m6A alternative-model per-position scores; comparison arm of the benchmark)
- **nanopolish** (Indexing and eventalign (--scale-events, -n) to produce the event TSV shared by both callers) — https://github.com/jts/nanopolish
- **bwa mem** (Aligns ONT fastq reads to the reference with -x ont2d; alternative aligner graphmap also supported)
- **scikit-learn** (Computes precision-recall curves and AUC per motif for the comparison)
- **samtools** (Sorts and indexes the BAM produced by bwa mem)

## Examples

```
mCaller.py -m GATC -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A; python make_bed.py -d <min_depth> -t 0.5
```

## Evaluation signals

- Per-motif PR-AUC values match the published Fig 4 comparison within reasonable tolerance: mCaller vs Tombo GATC 0.91 vs 0.83; AAGANNNNNCTC 0.71 vs 0.86; GAGNNNNNTCTT 0.90 vs 0.64
- All motif sites in the P. aeruginosa negative control are labelled unmethylated and treated exclusively as true negatives
- make_bed.py filtering applied consistently to both callers: minimum read depth (-d) and mod_threshold of 0.5 (minimum fraction of reads labelled methylated)
- Both callers consume the same nanopolish eventalign TSV produced with --scale-events and -n, ensuring feature comparability
- pr_auc_comparison.csv contains one PR-AUC row per tested motif (GATC, AAGANNNNNCTC, GAGNNNNNTCTT) for each method

## Limitations

- mCaller accuracy decreases at sequence contexts dissimilar to its training examples due to training biases; it notably failed at one S. aureus motif
- Tombo performance at non-2016-version behaviour is an open reproducibility question; the comparison assumes it still uses the same strategies as its 2016 version
- Both single-molecule methods can fail to distinguish methylation types (e.g. the GATCGVNY motif in S. aureus) and predicted lower methylation than combined PacBio and MeDIP-seq predictions
- Nanopolish does not support R10.4 flowcells, so the eventalign-based feature pipeline is limited to R9-era chemistries
- Non-motif site calls are method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios

## Evidence

- [discussion] Tombo and mCaller both showed variable performance across the motifs tested and predicted lower methylation than the combined predictions from PacBio and MeDIP-seq: "Tombo and mCaller both showed variable performance across the motifs tested and predicted lower methylation than the combined predictions from PacBio and MeDIP-seq"
- [methods] mCaller per-motif command with R9.5 model: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] nanopolish eventalign produces scaled event TSV: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
- [methods] make_bed.py threshold parameters: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [readme] mCaller README purpose statement: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
