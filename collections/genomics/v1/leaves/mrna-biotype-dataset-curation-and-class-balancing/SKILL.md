---
name: mrna-biotype-dataset-curation-and-class-balancing
description: Use when building a binary RNA-class classifier from raw nanopore DRS
  signals and you need biotype-verified labels, removal of ambiguous transcripts (e.g.
  pseudogenes), and a 50/50 class balance from an imbalanced read pool.
when_to_use_negative:
- The input reads are not direct RNA sequencing data with a 3'-end anchored signal
  (the class signal lives at the start of the DRS read)
- You need labels for RNA classes other than mRNA vs non-mRNA without rethinking pseudogene
  and low-expression handling, e.g. lncRNAs, whose accurate 3'-end identification
  the authors report as still challenging
- The dataset is already class-balanced and biotype-filtered; re-undersampling would
  discard majority-class data unnecessarily
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files with options --flowcell FLO-MIN106 --kit SQK-RNA002
    to obtain the read-level labels pipeline input
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE release 34 transcriptome with -ax map-ont
    --secondary=no to attach transcript/biotype annotations
- name: samtools (v1.10)
  role: Filters mappings with option -F 2324, removing reverse-strand, secondary,
    supplementary, and unmapped alignments
- name: BoostNano
  role: Removes the sequencing adapter and poly(A) tail from raw nanopore signals
    so the transcript-derived first 4 s can be extracted for classification
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:4b8533c768e26ff09364ee2614b1f82bea8ce6bf124dd93658b2b647ed981726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - PyTorch (v1.9.0)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# mRNA biotype dataset curation and class balancing

## Summary

Curates a balanced mRNA vs non-mRNA training/testing dataset for direct RNA sequencing (DRS) signal classifiers by basecalling and mapping MinION fast5 reads to GENCODE, labeling by biotype, balancing classes by undersampling, and trimming adapter/poly(A) so only the first 4 s of transcript signal remains. This yields the clean, class-balanced signal set on which models like RISER's CNN are trained.

## When to use

Use when building a binary RNA-class classifier from raw nanopore DRS signals and you need biotype-verified labels, removal of ambiguous transcripts (e.g. pseudogenes), and a 50/50 class balance from an imbalanced read pool.

## When NOT to use

- The input reads are not direct RNA sequencing data with a 3'-end anchored signal (the class signal lives at the start of the DRS read)
- You need labels for RNA classes other than mRNA vs non-mRNA without rethinking pseudogene and low-expression handling, e.g. lncRNAs, whose accurate 3'-end identification the authors report as still challenging
- The dataset is already class-balanced and biotype-filtered; re-undersampling would discard majority-class data unnecessarily

## Inputs

- MinION fast5 files from direct RNA sequencing (SQK-RNA002, FLO-MIN106)
- GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with biotype annotations
- BoostNano pretrained segmentation model

## Outputs

- Class-balanced (50/50) training set of mRNA/non-mRNA signals (1,073,720 signals in the RISER study)
- Reserved 20% test set (plus 20% of training reserved for tuning)
- Per-read adapter/poly(A)-trimmed signals, first 4 s, median-absolute-deviation normalized with outlier smoothing

## How to apply

Basecall model-development fast5 files with Guppy v4.0.14 (--flowcell FLO-MIN106 --kit SQK-RNA002), map to GENCODE release 34 (GRCh38.p13) with minimap2 v2.17 (-ax map-ont --secondary=no), and keep only high-confidence forward-strand primary alignments using samtools v1.10 with -F 2324. Assign reads to mRNA/non-mRNA classes from GENCODE biotypes, removing pseudogenes because they share sequences with coding transcripts and would blur the label boundary. Randomly split 80/20 into train/test (20% of training reserved for tuning), undersample the majority mRNA class to a 50/50 balance, then trim the sequencing adapter and poly(A) tail with BoostNano and extract the first 4 s of the remaining signal, normalizing by median absolute deviation with outlier smoothing. In the RISER study this produced a final training set of 1,073,720 signals used to compare CNN, ResNet, and TCN architectures.

## Related tools

- **Guppy (v4.0.14)** (Basecalls fast5 files with options --flowcell FLO-MIN106 --kit SQK-RNA002 to obtain the read-level labels pipeline input)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE release 34 transcriptome with -ax map-ont --secondary=no to attach transcript/biotype annotations)
- **samtools (v1.10)** (Filters mappings with option -F 2324, removing reverse-strand, secondary, supplementary, and unmapped alignments)
- **BoostNano** (Removes the sequencing adapter and poly(A) tail from raw nanopore signals so the transcript-derived first 4 s can be extracted for classification) — https://github.com/haotianteng/BoostNano

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- Only primary forward-strand alignments remain: filtering with samtools -F 2324 must have removed reverse-strand, secondary/supplementary, and unmapped reads
- Pseudogene transcripts are absent from both classes; mRNA/non-mRNA labels derive from GENCODE biotypes
- Class counts are 50/50 after undersampling the majority mRNA class
- Final signal count matches the study scale (~1,073,720 training signals at 4 s input length)
- Each retained signal has been adapter/poly(A)-trimmed by BoostNano and MAD-normalized with outlier smoothing

## Limitations

- Undersampling the majority mRNA class discards signal data that could benefit training of rare-class detection
- Pseudogenes must be excluded because shared sequences make their biotype labels ambiguous for signal classification
- The 4 s input length is a compromise between minimizing rejection latency and maximizing accuracy; the trade-off must be re-evaluated per application

## Evidence

- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] samtools high-confidence filtering: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [methods] BoostNano trimming of adapter and poly(A) tail: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [methods] mapping to GENCODE release 34 with minimap2: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [readme] BoostNano README: purpose of segmentation before basecalling: [evidence span withheld — anchor status: external_doc]
