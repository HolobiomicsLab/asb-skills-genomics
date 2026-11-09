---
name: pacbio-orthogonal-methylation-label-construction
description: Use when you need labeled methylated/unmethylated positions from PacBio-validated
  motif sites (e.g., E. coli GATC) to train or score ONT nanopore m6A classifiers
  on current-deviation features.
when_to_use_negative:
- You already have a trained .pkl model and only need per-read calling (use mCaller.py
  without --train, e.g., -m GATC -b A).
- The target context has no PacBio-confirmed motif sites — training labels cannot
  be constructed, and mCaller accuracy decreases at contexts dissimilar to training
  examples.
- The organism's motifs cannot be confidently assigned (e.g., C. neoformans and S.
  cerevisiae, where sites were not consistently detected across PacBio, nanopore,
  and MeDIP-seq).
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: mCaller
  role: Consumes the PacBio-derived positions file to train classifiers (--train,
    -p, -c) on current-deviation features; saves models as .pkl
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes the fastq against fast5 and runs eventalign --scale-events to produce
    the scaled event-level TSV from which features at labeled positions are extracted
  repo: https://github.com/jts/nanopolish
- name: SMRT analysis pipeline
  role: Provides the orthogonal PacBio calls (polymerase-kinetics t-test based) that
    define methylated and unmethylated labels at motif sites
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:1d322f4bee279b6357aaac062255ea76d86794aed9147a98da1640cb2f5f8164
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - SMRT analysis pipeline
  - bwa
  - mCaller
  - nanopolish
  - samtools
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pacbio-orthogonal-methylation-label-construction

## Summary

Builds ground-truth labels (chromosome, position, strand, methylated/unmethylated) from orthogonal PacBio SMRT calls at known methyltransferase motifs, so binary classifiers like mCaller's neural network can be trained and evaluated on nanopore current-deviation features.

## When to use

Use when you need labeled methylated/unmethylated positions from PacBio-validated motif sites (e.g., E. coli GATC) to train or score ONT nanopore m6A classifiers on current-deviation features.

## When NOT to use

- You already have a trained .pkl model and only need per-read calling (use mCaller.py without --train, e.g., -m GATC -b A).
- The target context has no PacBio-confirmed motif sites — training labels cannot be constructed, and mCaller accuracy decreases at contexts dissimilar to training examples.
- The organism's motifs cannot be confidently assigned (e.g., C. neoformans and S. cerevisiae, where sites were not consistently detected across PacBio, nanopore, and MeDIP-seq).

## Inputs

- ONT R9 nanopore reads (fastq plus fast5) basecalled with Albacore or another basecaller that saves event data
- Reference genome FASTA (e.g., E. coli MG1655 K12)
- Orthogonal PacBio SMRT calls of methylated and unmethylated positions at known motifs (GATC, AACNNNNNNGTGC)
- Positions file formatted as chromosome, position, strand, and label for training

## Outputs

- Trained classifier model files (.pkl, e.g., model_NN_6_m6A.pkl)
- Per-read methylation prediction files (*.eventalign.diffs.6) with per-read methylation probabilities
- Per-position methylation fractions and per-position accuracy against PacBio labels

## How to apply

Start from motif sites confirmed with orthogonal PacBio data generated for the ONT-sequenced strain — in E. coli MG1655 K12, the GATC and AACNNNNNNGTGC motifs — and format the labels as a space- or tab-separated positions file with chromosome, position, strand, and label, as mCaller's -p/--positions argument requires for --train. Index the Albacore-basecalled R9 fastq with nanopolish index, align reads to the reference with bwa mem -x ont2d followed by samtools sort, and run nanopolish eventalign with --scale-events to obtain scaled event-level TSVs that supply the current-deviation observations at each labeled position. Train per-classifier models with mCaller.py --train (e.g., -c NN, -c RF, -c NBC, -c LR) using the default six-variable (11-mer) context and skip_thresh 0, saving each model as a .pkl file. Evaluate by applying each .pkl model to a second same-strain dataset's eventalign TSV, aggregating per-read probabilities to per-position methylation fractions, and computing per-position accuracy against the PacBio labels.

## Related tools

- **mCaller** (Consumes the PacBio-derived positions file to train classifiers (--train, -p, -c) on current-deviation features; saves models as .pkl) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes the fastq against fast5 and runs eventalign --scale-events to produce the scaled event-level TSV from which features at labeled positions are extracted) — https://github.com/jts/nanopolish
- **SMRT analysis pipeline** (Provides the orthogonal PacBio calls (polymerase-kinetics t-test based) that define methylated and unmethylated labels at motif sites)

## Examples

```
python mCaller.py --train -p m6A_positions.txt -r reference.fasta -e reads.eventalign.tsv -f reads.fastq -c NN -b A
```

## Evaluation signals

- Positions file passes schema check: space- or tab-separated chromosome, position, strand, and label columns, restricted to PacBio-confirmed motif sites (GATC, AACNNNNNNGTGC in E. coli MG1655 K12).
- On a second same-strain E. coli dataset, per-position accuracy reproduces the reported ranking: neural network ~81.3% slightly above random forest ~80.8% (Fig 2b / Supplementary Table 1).
- Per-position accuracy across motifs falls within the reported 60–98% range.
- Each --train run with -c NN/RF/NBC/LR produces a .pkl model file; applying a model generates *.eventalign.diffs.6 per-read prediction files.

## Limitations

- Accuracy degrades at sequence contexts not covered by the training labels: training biases could decrease accuracy at contexts with lower similarity to the training examples.
- A model trained on E. coli motifs notably failed at one S. aureus motif, and both single-molecule methods can fail to distinguish methylation types (GATCGVNY motif in S. aureus).
- Non-motif site calls are method-specific false positives reflecting different noise sources for nanopore current deviations versus PacBio IPD ratios.

## Evidence

- [intro] Methylated/unmethylated labels derive from orthogonal PacBio motif confirmation: "using known motif sites confirmed with orthogonal PacBio data generated for ONT-sequenced E. coli DNA samples24, we identify sets of methylated and unmethylated positions"
- [readme] Required format of the labeled positions file for mCaller training: "file with a list of positions at which to classify bases (must be formatted as space- or tab-separated file with chromosome, position, strand, and label if training)"
- [methods] How a trained model is applied with the positions/motif and base arguments: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] Event-level TSV preparation that underlies the current-deviation features: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
