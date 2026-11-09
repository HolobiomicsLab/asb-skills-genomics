---
name: dna-barcode-ligation-library-preparation
description: Use when you need simulated barcoded nanopore reads with known ground-truth
  barcode identities to benchmark demultiplexing accuracy of tools like PRO or Guppy.
when_to_use_negative:
- Data are already real ONT reads with known sample-of-origin — no simulation is needed;
  proceed directly to demultiplexing
- You need PacBio simulated data — the pipeline targets ONT simulation, and the article
  notes 'no tests on PacBio data have been performed yet'
- You only need official ONT kit barcodes without custom barcode sets — the pipeline's
  built-in kits suffice without the custom barcode interface
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: deepsimulator 1.5
  role: Simulates the ONT nanopore sequencing process on the barcoded library, introducing
    insertion, deletion and substitution errors to yield FASTQ reads
  repo: https://github.com/liyu95/DeepSimulator
- name: ONT-sequencing-data-library-preparation-pipeline
  role: Fragments raw DNA, ligates user-supplied barcodes with flanking/adapter sequences
    (structure adapters-flanking-barcode-flanking-insert), and invokes DeepSimulator
  repo: https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- name: PRO
  role: Supplies the FPS-generated 24 bp barcode set (e.g., B9, size 2,292) ligated
    during simulated library preparation
  repo: https://github.com/rztongr/PRO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:b0b977c1e5d49ac6dc67fd6b75699571241536e51ccfc2693823915efe3d0128
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/liyu95/DeepSimulator
  tools:
  - Anaconda3
  - ONT-sequencing-data-library-preparation-pipeline
  - PRO
  - Python
  - deepsimulator 1.5
  - tqdm
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dna-barcode-ligation-library-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates Oxford Nanopore library preparation by ligating 24 bp barcodes and their flanking sequences onto raw DNA template reads in the structure adapters-flanking-barcode-flanking-insert, then generates ONT-like reads with DeepSimulator 1.5 to produce ground-truth-barcoded test data.

## When to use

Use when you need simulated barcoded nanopore reads with known ground-truth barcode identities to benchmark demultiplexing accuracy of tools like PRO or Guppy.

## When NOT to use

- Data are already real ONT reads with known sample-of-origin — no simulation is needed; proceed directly to demultiplexing
- You need PacBio simulated data — the pipeline targets ONT simulation, and the article notes 'no tests on PacBio data have been performed yet'
- You only need official ONT kit barcodes without custom barcode sets — the pipeline's built-in kits suffice without the custom barcode interface

## Inputs

- Raw DNA template reads in FASTA format (e.g., E. coli whole-genome shotgun sequences and lambda phage genome)
- Barcode sequences (e.g., PRO-generated 24 bp kit B9, 2,292 barcodes) in CSV format (Seq_ID | Reverse Sequence | Forward Sequence)
- Flanking sequences in FASTA format (upstream/downstream regions flanking the barcode)
- Adapter sequences in FASTA format

## Outputs

- Barcoded library sequences with structure adapters-flanking-barcode-flanking-insert per template read
- Simulated nanopore sequencing reads in FASTQ format (saved per raw sequence in a 'XXXX_lib_fastq' folder), each carrying a known ground-truth barcode label

## How to apply

Ligate the FPS-generated 24 bp barcode set (e.g., kit B9 of size 2,292) together with its flanking sequences onto the raw DNA template reads (e.g., 104 E. coli whole-genome shotgun sequences plus Enterobacteria phage lambda) using the multisample ONT-sequencing-data-library-preparation-pipeline. The library structure is adapters-flanking-barcode-flanking-insert, and custom barcode kits and flanking sequences are supplied as CSV and FASTA files respectively. Then run deepsimulator 1.5 on the barcoded library to produce FASTQ reads with ONT-like insertion, deletion and substitution errors. The ground-truth barcode assigned to each read during ligation becomes the reference label for downstream demultiplexing evaluation.

## Related tools

- **deepsimulator 1.5** (Simulates the ONT nanopore sequencing process on the barcoded library, introducing insertion, deletion and substitution errors to yield FASTQ reads) — https://github.com/liyu95/DeepSimulator
- **ONT-sequencing-data-library-preparation-pipeline** (Fragments raw DNA, ligates user-supplied barcodes with flanking/adapter sequences (structure adapters-flanking-barcode-flanking-insert), and invokes DeepSimulator) — https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- **PRO** (Supplies the FPS-generated 24 bp barcode set (e.g., B9, size 2,292) ligated during simulated library preparation) — https://github.com/rztongr/PRO

## Examples

```
python lib_make_v.1.py -i ./example/data_for_12_ETEC -f ./model/flanking_template/flanking2.fasta -k 7 -b ./model/barcode_kits/User_template.csv
```

## Evaluation signals

- Each simulated FASTQ read carries a ground-truth barcode identity recorded during ligation, enabling downstream accuracy computation against PRO-dex or Guppy assignments
- Library structure per read should match adapters-flanking-barcode-flanking-insert with the correct 24 bp barcode inserted
- Simulated reads exhibit ONT-like insertion, deletion and substitution error profiles as generated by deepsimulator 1.5
- Output folder contains per-raw-sequence FASTQ files (e.g., in 'XXXX_lib_fastq'), plus intermediate folders such as 'results_loading', 'dis_create', 'barcode_kits' and 'lib_results'
- Demultiplexing PRO-dex on the resulting B9 data should reach approximately 98.29% accuracy (vs. 93.98% for Guppy), matching the published benchmark

## Limitations

- Sequencing data are simulated; DeepSimulator 1.5 error models may not fully capture real ONT signal-level characteristics
- The pipeline supports only ONT simulation; no PacBio tests have been performed
- Custom kits require flanking and barcode files in specific FASTA/CSV formats, and single-end ligation requires flanking sequences ordered as upstream then downstream region

## Evidence

- [other] Ligate the FPS 24 bp barcode set with flanking sequences onto raw DNA template reads in the structure (adapters-flanking-barcode-flanking-insert).: [evidence span withheld — anchor status: not_found]
- [readme] This project is used for library preparation before Oxford nanopore sequencing. It supports barcode library construction.: "This project is used for library preparation before Oxford nanopore sequencing. 
It supports barcode library construction."
- [results] we simulated library preparation and sequencing for ONT via a multisample sequencing simulator: "we simulated library preparation and sequencing for ONT via a multisample sequencing simulator"
- [readme] The sequencing data in examples folder file is generated by the simulator deepsimulator 1.5.: "The sequencing data in examples folder file is generated by the simulator deepsimulator 1.5."
