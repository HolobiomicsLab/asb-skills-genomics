---
name: ont-read-simulation-with-deepsimulator
description: Use when you need simulated ONT sequencing data for benchmarking barcode
  kits or demultiplexing tools — e.g., ligating custom or official ONT kits onto reference
  genomes and producing FASTQ reads with realistic error profiles.
when_to_use_negative:
- When real ONT read data are already available — simulation should not replace validation
  on experimental reads, and PRO's accuracy claims are simulation-based
- 'When the target platform is PacBio: the article states no tests on PacBio data
  have been performed, so simulation parameters are tuned for ONT only'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deepsimulator 1.5
  role: Simulates the Oxford Nanopore sequencing process on prepared libraries to
    produce signal and FASTQ data
  repo: https://github.com/lykaust15/DeepSimulator
- name: ONT-sequencing-data-library-preparation-pipeline
  role: Multisample simulator that fragments raw sequences, ligates barcode kits with
    flanking sequences, and invokes DeepSimulator
  repo: https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- name: Anaconda2 / Miniconda2
  role: Prerequisite environment for running DeepSimulator
  repo: https://www.anaconda.com/distribution/
- name: PRO
  role: Source of the custom 96-barcode, 24 bp kit that is ligated onto reads before
    simulation
  repo: https://github.com/rztongr/PRO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:28631eaa568d41e88dd5fdd50679f81086108a12ad32385bab6f9ba9beefe9e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/lykaust15/DeepSimulator
  tools:
  - Anaconda2 / Miniconda2
  - Anaconda3
  - ONT-sequencing-data-library-preparation-pipeline
  - PRO
  - Python
  - deepsimulator 1.5
  - tqdm
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# ONT read simulation with DeepSimulator

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates Oxford Nanopore sequencing reads by ligating a barcode kit onto raw DNA sequences (e.g., E. coli WGS and lambda phage reads) via a multisample library-preparation pipeline, then running DeepSimulator 1.5 to generate FASTQ read data for benchmarking demultiplexing tools.

## When to use

Use when you need simulated ONT sequencing data for benchmarking barcode kits or demultiplexing tools — e.g., ligating custom or official ONT kits onto reference genomes and producing FASTQ reads with realistic error profiles.

## When NOT to use

- When real ONT read data are already available — simulation should not replace validation on experimental reads, and PRO's accuracy claims are simulation-based
- When the target platform is PacBio: the article states no tests on PacBio data have been performed, so simulation parameters are tuned for ONT only

## Inputs

- Reference genome FASTA files (e.g., 104 whole-genome shotgun sequences of E. coli and the complete Enterobacteria phage lambda genome)
- Barcode kit CSV file (built-in ONT kits such as EXP-NBD104, SQK-16S024, EXP-PBC096, or a custom kit CSV with Seq_ID, Reverse Sequence, Forward Sequence columns)
- Flanking sequence FASTA file (required for custom kits)
- Kit selection flag -k and adapter FASTA (optional custom adapters)

## Outputs

- Simulated ONT electrical signal files in txt format
- Simulated ONT sequencing reads in FASTQ format (per raw sequence, in the output *_lib_fastq folder)
- Intermediate library-preparation folders (results_loading, dis_create, barcode_kits, lib_results)

## How to apply

First run the multisample library preparation pipeline (JustLeeee/ONT-sequencing-data-library-preparation-pipeline, e.g. `python lib_make_v.1.py -i input.fasta -k <kit_index> -b <barcode_kit.csv>`) to fragment reference sequences, attach flanking/adapter sequences, and ligate the chosen barcode kit (official kits like EXP-NBD104, SQK-16S024, EXP-PBC096 or custom PRO barcodes supplied as a CSV with Seq_ID/Reverse Sequence/Forward Sequence columns). Then invoke DeepSimulator 1.5 (via `python ./to_nanopore_seq.py -i <input_dir> -o <output_dir>`) to simulate the nanopore sequencing process, yielding electrical signal files (txt) and FASTQ reads stored per raw sequence in the output `*_lib_fastq` folder. DeepSimulator requires Anaconda2/Miniconda2 and must be installed via its own `install.sh`. The resulting simulated reads are then demultiplexed with the tools under comparison (e.g., PRO and Guppy 6.5.7) to measure demultiplexing accuracy.

## Related tools

- **deepsimulator 1.5** (Simulates the Oxford Nanopore sequencing process on prepared libraries to produce signal and FASTQ data) — https://github.com/lykaust15/DeepSimulator
- **ONT-sequencing-data-library-preparation-pipeline** (Multisample simulator that fragments raw sequences, ligates barcode kits with flanking sequences, and invokes DeepSimulator) — https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- **Anaconda2 / Miniconda2** (Prerequisite environment for running DeepSimulator) — https://www.anaconda.com/distribution/
- **PRO** (Source of the custom 96-barcode, 24 bp kit that is ligated onto reads before simulation) — https://github.com/rztongr/PRO

## Examples

```
python lib_make_v.1.py -i ./example/data_for_96_ETEC -k 3 && python ./to_nanopore_seq.py -i ./lib_out/ -o ./sim_out/
```

## Evaluation signals

- Output folder must contain electrical signal files in txt format and FASTQ reads per raw sequence in the *_lib_fastq folder
- Simulated reads must carry the correct flanking-barcode-flanking structure for every barcode in the ligated kit (e.g., all 96 PRO barcodes or all barcodes of B1/B2/B3)
- Downstream demultiplexing of official kits (EXP-NBD104, SQK-16S024, EXP-PBC096) with Guppy 6.5.7 should yield the reported high accuracies (e.g., 100.00%/99.75% for B1, 99.71%/99.21% for B2, 99.66%/98.00% for B3), confirming the simulation is realistic
- DeepSimulator version must be 1.5, matching the version the article used to generate its example data
- Number of simulated read files/reads should correspond to the input reference sequences and per-barcode read counts (e.g., 9,600 reads for the 96-barcode kit comparison)

## Limitations

- The library-preparation pipeline requires Anaconda2 or Miniconda2 (Python 2 environment) specifically for DeepSimulator, separate from the Python 3.7+ environment for the rest of the pipeline
- Simulated data may not capture all real ONT error modes; conclusions (e.g., PRO outperforming Guppy by ~4.31–4.34%) are grounded in simulation only
- No tests on PacBio data have been performed; simulation and demultiplexing results apply to ONT data
- DeepSimulator parameters can only be changed by modifying the to_nanopore_seq.py script directly

## Evidence

- [readme] to simulate the Oxford nanopore sequencing process to obtain sequencing datas: [evidence span withheld — anchor status: external_doc]
