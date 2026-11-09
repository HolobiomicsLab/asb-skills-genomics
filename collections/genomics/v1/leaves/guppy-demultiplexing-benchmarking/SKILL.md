---
name: guppy-demultiplexing-benchmarking
description: Use when comparing demultiplexing accuracy of an ONT barcode caller (e.g.
  PRO) against Guppy on simulated nanopore reads, or when validating a newly designed
  barcode kit against official ONT kits B1–B3.
when_to_use_negative:
- Input is PacBio sequencing data — the authors note no tests on PacBio data have
  been performed, so Guppy/ONT-kit benchmarks do not transfer
- The goal is barcode kit design or expansion (e.g. up to 2,292 barcodes) rather than
  head-to-head demultiplexing accuracy comparison
- Reads were produced by real ONT flow cells rather than the deepsimulator 1.5 / multisample
  simulator pipeline, since the reported accuracy deltas are established only on simulated
  data
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: Guppy
  role: Official ONT demultiplexing baseline run with barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg,
    and barcode_arrs_pcr96.cfg configs
- name: PRO
  role: Comparator tool run in demultiplexing mode (-i, -f, -o, -t) with narrowed
    flanking alignment ranges sequence'[35:77] and sequence'[35+startf:120]
  repo: https://github.com/rztongr/PRO
- name: ONT-sequencing-data-library-preparation-pipeline
  role: Multisample simulator that ligates barcode kits (built-in EXP-NBD104, SQK-16S024,
    EXP-PBC096) onto raw sequences before sequencing simulation
  repo: https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- name: deepsimulator
  role: Version 1.5 simulator of the Oxford nanopore sequencing process invoked via
    to_nanopore_seq.py to generate FASTQ sequencing results
  repo: https://github.com/liyu95/DeepSimulator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:48215054432867d194dce2389f3d88dd8c5a7dc7cb8ac59a04b13704e617845f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/rztongr/PRO
  tools:
  - Anaconda3
  - Guppy
  - ONT-sequencing-data-library-preparation-pipeline
  - PRO
  - Python
  - deepsimulator
  - tqdm
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# guppy-demultiplexing-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks demultiplexing accuracy of Guppy 6.5.7 (official ONT barcode configs) against PRO on simulated ONT reads, comparing PRO-generated 96-barcode, 24 bp kits with official kits B1–B3. Used to validate that same-size PRO kits outperform Guppy by ~4.34% on average while PRO underperforms on official kits.

## When to use

Use when comparing demultiplexing accuracy of an ONT barcode caller (e.g. PRO) against Guppy on simulated nanopore reads, or when validating a newly designed barcode kit against official ONT kits B1–B3.

## When NOT to use

- Input is PacBio sequencing data — the authors note no tests on PacBio data have been performed, so Guppy/ONT-kit benchmarks do not transfer
- The goal is barcode kit design or expansion (e.g. up to 2,292 barcodes) rather than head-to-head demultiplexing accuracy comparison
- Reads were produced by real ONT flow cells rather than the deepsimulator 1.5 / multisample simulator pipeline, since the reported accuracy deltas are established only on simulated data

## Inputs

- Simulated ONT FASTQ reads with ligated barcode kits (E. coli WGS and lambda phage templates)
- PRO-generated 96-barcode, 24 bp kit (select_barcode.txt)
- Official ONT barcode kits EXP-NBD104 (B1), SQK-16S024 (B2), EXP-PBC096 (B3)
- Guppy 6.5.7 official config files: barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg
- Flanking-sequence fastq file used in library preparation

## Outputs

- demux_accuracy_comparison.csv with demultiplexing accuracies of PRO and Guppy 6.5.7 per barcode kit
- PRO demultiplexing results in the dex_result output folder
- Per-kit accuracy table for B1/B2/B3 official kits and the PRO-generated kit

## How to apply

Simulate library preparation and sequencing by ligating each barcode kit (the PRO 96-barcode 24 bp kit and official EXP-NBD104, SQK-16S024, EXP-PBC096) onto E. coli WGS and Enterobacteria phage lambda reads via the multisample sequencing simulator (lib_make_v.1.py) and deepsimulator 1.5, producing FASTQ reads. Demultiplex the same simulated reads with two tools: Guppy 6.5.7 using its official config files barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, and barcode_arrs_pcr96.cfg, and PRO in demultiplexing mode (-i input directory, -f flanking fastq, -o output directory, -t threads), narrowing the flanking alignment ranges to sequence'[35:77] and sequence'[35+startf:120]. Compile per-kit accuracies for both tools into demux_accuracy_comparison.csv. Judge success by reproducing the reported pattern: for same-size (96-barcode, 24 bp) kits PRO averages 4.34% higher accuracy (up to 4.73% higher), while on official kits B1/B2/B3 Guppy matches or exceeds PRO.

## Related tools

- **Guppy** (Official ONT demultiplexing baseline run with barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, and barcode_arrs_pcr96.cfg configs)
- **PRO** (Comparator tool run in demultiplexing mode (-i, -f, -o, -t) with narrowed flanking alignment ranges sequence'[35:77] and sequence'[35+startf:120]) — https://github.com/rztongr/PRO
- **ONT-sequencing-data-library-preparation-pipeline** (Multisample simulator that ligates barcode kits (built-in EXP-NBD104, SQK-16S024, EXP-PBC096) onto raw sequences before sequencing simulation) — https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- **deepsimulator** (Version 1.5 simulator of the Oxford nanopore sequencing process invoked via to_nanopore_seq.py to generate FASTQ sequencing results) — https://github.com/liyu95/DeepSimulator

## Examples

```
guppy_demultiplexer --config barcode_arrs_pcr96.cfg -i simulated_reads/ -o demux_out/  # repeat with barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg
```

## Evaluation signals

- demux_accuracy_comparison.csv contains paired PRO and Guppy 6.5.7 accuracies for each of the four kits (PRO 96-barcode kit, B1, B2, B3)
- For same-size (96-barcode, 24 bp) kits, PRO's accuracy exceeds Guppy's by ~4.34% on average and up to 4.73% on the largest gap
- For official kits B1–B3, Guppy matches or exceeds PRO (validation pattern: PRO underperforms Guppy on B1, B2, and B3)
- Both tools reach superior and comparable accuracy when the PRO kit size equals the official ONT kit size (96 barcodes, 24 bp)
- Guppy must be run with the official config files matching each kit (barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg), not a custom config

## Limitations

- PRO underperforms Guppy on official kits B1, B2, B3 because the flanking sequences are similar to the barcode and PRO cannot accurately extract them
- PRO also underperforms on official kits because ONT's design strategy for B1–B3 does not ensure the probability divergence between barcodes is large enough
- Accuracy figures are established via simulation (deepsimulator 1.5 and the multisample library-preparation pipeline), not real flow-cell runs

## Evidence
