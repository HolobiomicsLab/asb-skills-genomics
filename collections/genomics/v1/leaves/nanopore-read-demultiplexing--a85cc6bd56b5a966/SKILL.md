---
name: nanopore-read-demultiplexing
description: Use when nanopore FASTQ/FASTA reads have been ligated with a PRO-designed
  barcode kit and each read must be assigned to its source barcode, especially when
  comparing accuracy against Guppy or edit-distance assignment.
when_to_use_negative:
- Reads were barcoded with official ONT kits B1 (EXP-NBD104), B2 (SQK-16S024), or
  B3 (EXP-PBC096) whose flanking sequences resemble the barcodes — PRO may underperform
  Guppy because similar flanking sequences prevent accurate barcode extraction
- Data are PacBio reads — PRO has not been tested on PacBio data
- The goal is generating or selecting a new barcode set rather than assigning reads
  to existing barcodes
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: PRO
  role: Extracts corrupted barcodes from reads via edlib flanking-sequence alignment
    and assigns them by probability divergence (PRO-dex)
  repo: https://github.com/renztr/PRO.git
- name: Guppy
  role: Official ONT demultiplexing baseline using the same barcode kit for accuracy
    comparison
- name: edlib
  role: Alignment library used to map rear flanking sequences onto narrowed read ranges
    to extract corrupted barcodes
- name: deepsimulator
  role: Simulates ONT sequencing to generate the FASTQ reads demultiplexed in the
    examples
  repo: https://github.com/liyu95/DeepSimulator
- name: tqdm
  role: Progress-bar dependency for the PRO demultiplexing scripts
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:1ba5b2eb4b87870f788ca33fc6864d615673c6b1b5a0b46462f27aff6e83b1b3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/renztr/PRO.git
  tools:
  - Guppy
  - PRO
  - Python
  - deepsimulator
  - edlib
  - tqdm
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nanopore-read-demultiplexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Demultiplex corrupted DNA barcodes from simulated or real nanopore reads using PRO's PRO-dex strategy, which extracts corrupted barcodes via edlib alignment of flanking sequences and assigns them by probability divergence, outperforming Guppy's edit-distance-based demultiplexing on large PRO-designed barcode kits.

## When to use

Use when nanopore FASTQ/FASTA reads have been ligated with a PRO-designed barcode kit and each read must be assigned to its source barcode, especially when comparing accuracy against Guppy or edit-distance assignment.

## When NOT to use

- Reads were barcoded with official ONT kits B1 (EXP-NBD104), B2 (SQK-16S024), or B3 (EXP-PBC096) whose flanking sequences resemble the barcodes — PRO may underperform Guppy because similar flanking sequences prevent accurate barcode extraction
- Data are PacBio reads — PRO has not been tested on PacBio data
- The goal is generating or selecting a new barcode set rather than assigning reads to existing barcodes

## Inputs

- Directory of nanopore sequencing FASTA/FASTQ reads (e.g., DeepSimulator 1.5 simulated reads)
- Barcode kit file select_barcode.txt (e.g., PRO-generated 24-mer kit of up to 2,292 barcodes)
- FASTQ file containing the flanking sequences used in library preparation
- Optionally, Guppy 6.5.7 with the same barcode kit configuration for baseline comparison

## Outputs

- Per-read barcode assignment results in the dex_result folder
- Demultiplexing accuracy values per tool (fraction of reads assigned to the correct barcode)
- Two-row accuracy comparison table (e.g., accuracy_comparison.csv) for PRO vs Guppy

## How to apply

Run PRO's demultiplexing.sh: for each read, align the rear top and rear bottom flanking sequences (supplied in a fastq file) to the narrowed ranges sequence'[35:77] and sequence'[35+startf:120] using edlib 1.3.9 to extract the corrupted barcode s'. Then assign s' to the kit barcode s0 that maximizes pro(s',s0) over the barcodes in select_barcode.txt (probability-divergence assignment, PRO-dex). As a baseline, demultiplex the identical reads with Guppy 6.5.7 using the same barcode kit, and compute demultiplexing accuracy as the fraction of reads assigned to the correct barcode. On the 2,292-barcode (2,922-size kit) PRO design with DeepSimulator 1.5-simulated reads, expect PRO to reach 98.29% accuracy, about 4.31 percentage points above Guppy; when the kit size matches the official ONT kit size, the two tools should be comparable.

## Related tools

- **PRO** (Extracts corrupted barcodes from reads via edlib flanking-sequence alignment and assigns them by probability divergence (PRO-dex)) — https://github.com/renztr/PRO.git
- **Guppy** (Official ONT demultiplexing baseline using the same barcode kit for accuracy comparison)
- **edlib** (Alignment library used to map rear flanking sequences onto narrowed read ranges to extract corrupted barcodes)
- **deepsimulator** (Simulates ONT sequencing to generate the FASTQ reads demultiplexed in the examples) — https://github.com/liyu95/DeepSimulator
- **tqdm** (Progress-bar dependency for the PRO demultiplexing scripts)

## Examples

```
bash demultiplexing.sh
```

## Evaluation signals

- PRO's demultiplexing accuracy on the 2,292-barcode kit reaches 98.29% on DeepSimulator 1.5 simulated reads
- PRO exceeds Guppy 6.5.7 by 4.31 percentage points (Guppy baseline 93.98%) on the largest kit
- Accuracy is computed as the fraction of reads assigned to the correct source barcode
- When the kit size equals the official ONT kit size, PRO and Guppy accuracies should be comparable (both superior)
- All reads in the dex_result output folder carry exactly one barcode assignment from select_barcode.txt

## Limitations

- PRO underperforms Guppy on official ONT kits B1, B2, and B3 because the flanking sequences are similar to the barcodes and PRO cannot accurately extract them
- ONT's design strategy for B1–B3 does not ensure large probability divergence between barcodes, which disadvantages PRO
- No tests on PacBio data have been performed, although PRO could in principle be employed for both ONT and PacBio data

## Evidence
