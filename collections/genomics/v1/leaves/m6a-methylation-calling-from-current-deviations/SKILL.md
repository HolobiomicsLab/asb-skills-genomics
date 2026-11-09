---
name: m6a-methylation-calling-from-current-deviations
description: Use when you have ONT fast5/fastq data (basecalled with Albacore or a
  basecaller retaining event data), a reference assembly, and need per-read and per-position
  m6A calls at known or suspected methyltransferase motifs.
when_to_use_negative:
- The adenine contexts of interest are very dissimilar to the E. coli training motifs
  — training biases can decrease accuracy at untrained contexts, as seen in the notable
  mCaller failure at one S. aureus motif
- You need to distinguish methylation types at ambiguous motifs (e.g. the GATCGVNY
  motif in S. aureus), which both single-molecule methods failed to separate
- Nanopolish R10.4 flowcell data — nanopolish does not support R10.4 for methylation
  calling
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: mCaller
  role: Neural-network classifier that calls m6A per position from deviations between
    measured and expected nanopore currents, per motif or position list
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes fast5/fastq and produces the scaled event-alignment TSV that mCaller
    consumes
  repo: https://github.com/jts/nanopolish
- name: bwa mem
  role: Aligns ONT fastq reads to the reference assembly (with -x ont2d) before event
    alignment
- name: samtools
  role: Sorts and indexes the BAM produced by bwa mem
- name: mCaller_analysis_scripts
  role: Analysis scripts (make_bed.py and aggregation) for the Nature Communications
    paper workflow
  repo: https://github.com/al-mcintyre/mCaller_analysis_scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:e28e0a52a21609b094ac3d5d367413c4e15c7887896a2f4dc92e8421e335cce7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - bwa mem
  - mCaller
  - mCaller_analysis_scripts
  - nanopolish
  - pandas
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# m6a-methylation-calling-from-current-deviations

## Summary

Detect N6-methyladenine (m6A) in Oxford Nanopore data with mCaller by classifying deviations between measured and expected ionic current as adenine passes through the pore, then summarize per-read calls into per-position methylation predictions.

## When to use

Use when you have ONT fast5/fastq data (basecalled with Albacore or a basecaller retaining event data), a reference assembly, and need per-read and per-position m6A calls at known or suspected methyltransferase motifs.

## When NOT to use

- The adenine contexts of interest are very dissimilar to the E. coli training motifs — training biases can decrease accuracy at untrained contexts, as seen in the notable mCaller failure at one S. aureus motif
- You need to distinguish methylation types at ambiguous motifs (e.g. the GATCGVNY motif in S. aureus), which both single-molecule methods failed to separate
- Nanopolish R10.4 flowcell data — nanopolish does not support R10.4 for methylation calling

## Inputs

- ONT fastq reads plus fast5 signal files and sequencing_summary.txt (basecalled with Albacore or another basecaller saving event data)
- Reference sequence fasta (e.g. strain assembly)
- Nanopolish scaled event-alignment TSV
- Pre-trained mCaller model file (e.g. r95_twobase_model_NN_6_m6A.pkl)
- Target motif string or positions.txt (chromosome, position, strand, label if training)

## Outputs

- Per-read methylation probability classifications per position from mCaller.py
- Per-position methylation prediction bed file from make_bed.py
- Species-by-motif fraction-of-sites-methylated table (e.g. fig3a_species_motif_percent_methylated.csv via pandas/pysam aggregation)

## How to apply

Index template-strand reads with 'nanopolish index -d <fast5 directory> -s sequencing_summary.txt <filename>.fastq', align fastq to the reference with 'bwa mem -x ont2d' (graphmap also supported), sort/index with samtools, then run 'nanopolish eventalign -t <num_threads> --scale-events -n' to produce a scaled event-alignment TSV. Run 'mCaller.py -m <motif, e.g. GATC> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <eventalign.tsv> -f <fastq> -b A' — using a neural-network pickle model trained on E. coli MG1655 K12 methylated/unmethylated positions — to obtain per-read methylation probabilities, optionally tuning -s/--skip_thresh and -q/--qual_thresh. Run make_bed.py with -d (minimum read depth for inclusion) and -t 0.5 (--mod_threshold, minimum fraction of per-read observations labeled methylated) to generate a per-position bed file. Models can also be extended to confirm suspected methyltransferase target motifs at untrained contexts, e.g. across the eight ZymoBIOMICS strains where most PacBio-predicted motifs showed clear enrichment.

## Related tools

- **mCaller** (Neural-network classifier that calls m6A per position from deviations between measured and expected nanopore currents, per motif or position list) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes fast5/fastq and produces the scaled event-alignment TSV that mCaller consumes) — https://github.com/jts/nanopolish
- **bwa mem** (Aligns ONT fastq reads to the reference assembly (with -x ont2d) before event alignment)
- **samtools** (Sorts and indexes the BAM produced by bwa mem)
- **mCaller_analysis_scripts** (Analysis scripts (make_bed.py and aggregation) for the Nature Communications paper workflow) — https://github.com/al-mcintyre/mCaller_analysis_scripts

## Examples

```
mCaller.py -m GATC -r reference.fasta -d r95_twobase_model_NN_6_m6A.pkl -e sample.eventalign.tsv -f sample.fastq -b A
```

## Evaluation signals

- Per-position accuracy falls in the reported 60–98% range at most motifs, validated against PacBio-confirmed motif sites in E. coli MG1655 K12
- Fraction of PacBio-predicted motif sites classified as methylated per species/motif shows clear enrichment in ZymoBIOMICS strains, confirming methyltransferase target motifs
- make_bed.py output respects the -d minimum read depth and -t 0.5 mod_threshold filters (bed contains only positions meeting depth and methylation-fraction cutoffs)
- Event-alignment TSV is produced with --scale-events so current expectations match the sequencer; check TSV completeness before running mCaller

## Limitations

- Method has limitations for sequence contexts on which the model has not been trained; training biases could decrease accuracy at contexts with lower similarity to the training examples
- mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif
- m6A is detected with lower per-read accuracy (~70%) than m5C (upwards of 80%) in nanopore data
- Both single-molecule sequencing methods can fail to distinguish between types of methylation, as seen for the GATCGVNY motif in S. aureus
- Non-motif site calls are method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios

## Evidence

- [abstract] deviations between measured and expected current values: "using deviations between measured and expected current values as each adenine travels through a pore"
- [methods] mCaller workflow commands: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] mod_threshold definition: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [discussion] accuracy across motifs: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [discussion] nanopore per-read accuracy by modification type: "nanopore sequencing can detect m5C with per-read accuracy of upwards of 80% but m6A with lower accuracy (~70%)"
- [readme] mCaller dependencies: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
- [abstract] motif extension finding: "can be extended to detect known or confirm suspected methyltransferase target motifs based on predictions of methylation at untrained contexts"
