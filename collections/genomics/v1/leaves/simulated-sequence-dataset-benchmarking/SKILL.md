---
name: simulated-sequence-dataset-benchmarking
description: Use when comparing segmentation behavior of MEM-based aligners (FMAlign2
  vs FMAlign) across controlled similarity gradients and MEM coverage values, or when
  sweeping the minimum MEM length (-l) on ultra-long nucleotide FASTA inputs.
when_to_use_negative:
- The goal is alignment accuracy (SP score) or runtime benchmarking rather than segment-count
  comparison of MEM-based segmentation
- Input sequences are not nucleotide FASTA (FMAlign2 targets ultra-long nucleotide
  sequences)
- You need protein multiple sequence alignment or short-read data, which is outside
  FMAlign2's ultra-long nucleotide scope
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FMAlign2
  role: Aligner under test, run in global and local modes with -l minimum MEM length
    to measure segment production
  repo: https://github.com/malabz/FMAlign2
- name: FMAlign
  role: Baseline aligner run with identical k=39 settings to obtain reference segment
    counts
- name: MAFFT
  role: Default MSA backend invoked by FMAlign2 for the subMSA steps during each run
  repo: https://mafft.cbrc.jp/alignment/software/
- name: conda
  role: Installation channel for FMAlign2 and its MSA tool dependencies ('conda install
    -c malab fmalign2')
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:fef2579ed4f4b79824dc57652ddfec40e3470f83d7cac303cdca07aff1400db7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign
  - FMAlign2
  - MAFFT
  - conda
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# simulated-sequence-dataset-benchmarking

## Summary

Benchmark FMAlign2's global and local segmentation modes against FMAlign by counting the number of segments produced on simulated mitochondrial-like datasets (70-99% similarity, MEM coverage c = 0.4-0.8) and via an -l parameter sweep on the Human Mitochondrial Genome dataset with minimum MEM length k=39 for parity with FMAlign.

## When to use

Use when comparing segmentation behavior of MEM-based aligners (FMAlign2 vs FMAlign) across controlled similarity gradients and MEM coverage values, or when sweeping the minimum MEM length (-l) on ultra-long nucleotide FASTA inputs.

## When NOT to use

- The goal is alignment accuracy (SP score) or runtime benchmarking rather than segment-count comparison of MEM-based segmentation
- Input sequences are not nucleotide FASTA (FMAlign2 targets ultra-long nucleotide sequences)
- You need protein multiple sequence alignment or short-read data, which is outside FMAlign2's ultra-long nucleotide scope

## Inputs

- Simulated mitochondrial-like nucleotide sequence datasets (70-99% similarity, MEM coverage c = 0.4-0.8) as FASTA files from Zenodo record 10435770
- Human Mitochondrial Genome (mt) dataset FASTA file
- k=39 minimum MEM length setting for FMAlign/FMAlign2 parity
- FMAlign binary for baseline segment counts

## Outputs

- segment_counts.csv with one row per (tool, mode, similarity, c) grid cell plus every -l sweep value, recording segment counts per run
- Non-empty aligned output FASTA files from every FMAlign2 and FMAlign run

## How to apply

Retrieve the simulated mitochondrial-like sequence datasets and the Human Mitochondrial Genome (mt) FASTA files from the Zenodo deposit (zenodo.org/records/10435770). Install FMAlign2 via conda ('conda install -c malab fmalign2'), which bundles the default MAFFT MSA backend. Run FMAlign2 in global mode on each simulated dataset with 'FMAlign2 -i <input.fasta> -o <output.fasta>', setting the minimum MEM length to k=39 for parity with FMAlign, then repeat across the full grid of 70-99% sequence similarities and MEM coverage values c = 0.4 and c = 0.8 in local mode. Run FMAlign on the same datasets with identical k=39 settings to obtain baseline segment counts. Sweep the '-l' minimum-MEM-length parameter (default 30; lower it below 30 when too few MEMs are found) on the mt dataset, recording segments per run. Compile tool, mode, similarity, coverage c, and -l value with segment counts into segment_counts.csv.

## Related tools

- **FMAlign2** (Aligner under test, run in global and local modes with -l minimum MEM length to measure segment production) — https://github.com/malabz/FMAlign2
- **FMAlign** (Baseline aligner run with identical k=39 settings to obtain reference segment counts)
- **MAFFT** (Default MSA backend invoked by FMAlign2 for the subMSA steps during each run) — https://mafft.cbrc.jp/alignment/software/
- **conda** (Installation channel for FMAlign2 and its MSA tool dependencies ('conda install -c malab fmalign2'))

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 39
```

## Evaluation signals

- segment_counts.csv contains one row per (tool, mode, similarity, c) grid cell plus every -l sweep value on the mt dataset
- Each run produced a non-empty output FASTA file
- FMAlign2 global and local modes produce substantially more segments than FMAlign on the simulated datasets and the mt -l sweep
- Local mode generally yields more segments than global mode, and both FMAlign2 modes converge at similarity extremes (70% and 99%)
- Segment counts are compiled with the matching tool, mode, similarity, coverage c, and -l value for each run

## Limitations

- Both FMAlign2 modes struggle at exceptionally low sequence similarity
- If too few MEMs are found, the -l minimum MEM length (default 30) must be lowered below 30
- The -f fast MEM filtering mode runs faster but finds fewer MEMs at the cost of sensitivity

## Evidence

- [abstract] Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy"
- [methods] FMAlign2 run command from methods: "FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]"
- [readme] -l parameter default and tuning guidance: "If too few MEMs are found, try lowering `-l` (e.g., below 30)."
- [readme] Zenodo dataset availability: "Source code and datasets are available at https://github.com/malabz/FMAlign2 and https://zenodo.org/records/10435770."
