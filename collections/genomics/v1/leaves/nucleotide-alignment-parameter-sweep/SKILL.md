---
name: nucleotide-alignment-parameter-sweep
description: Use when comparing how many segments an MSA partitioning tool (FMAlign2
  vs FMAlign) produces across similarity levels, MEM coverage c values, or minimum-MEM-length
  (-l) settings on nucleotide FASTA inputs.
when_to_use_negative:
- The goal is alignment accuracy (SP score) rather than segmentation behavior — use
  the sp.py evaluation workflow instead of counting segments
- Input sequences are short reads or already highly similar fragments where segmentation
  count is not the variable of interest
- The comparison target is a tool other than FMAlign/FMAlign2 with no segment-partitioning
  output, making segment counts incomparable
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: FMAlign2
  role: Primary aligner run in global and local modes with suffix-array MEM segmentation;
    -l sets minimum MEM length (default 30), -p selects the MSA backend
  repo: https://github.com/malabz/FMAlign2
- name: FMAlign
  role: Baseline segmenter run with identical k=39 minimum-MEM settings to compare
    segment counts
- name: MAFFT
  role: Default MSA backend invoked by FMAlign2 for the sub-MSA of each segment
  repo: https://mafft.cbrc.jp/alignment/software/
- name: conda
  role: Installation channel for FMAlign2 ('conda install -c malab fmalign2') and
    its MSA dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:590d1686db0487c15859d290c664ada2780ec819a409e3251f48afed25a66eb3
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nucleotide-alignment-parameter-sweep

## Summary

Reproduce FMAlign2's segment-count benchmarking by sweeping its global and local modes (MEM minimum length -l, coverage c) against FMAlign on simulated mitochondrial-like datasets and the Human Mitochondrial Genome (mt) dataset, compiling per-run segment counts into a CSV.

## When to use

Use when comparing how many segments an MSA partitioning tool (FMAlign2 vs FMAlign) produces across similarity levels, MEM coverage c values, or minimum-MEM-length (-l) settings on nucleotide FASTA inputs.

## When NOT to use

- The goal is alignment accuracy (SP score) rather than segmentation behavior — use the sp.py evaluation workflow instead of counting segments
- Input sequences are short reads or already highly similar fragments where segmentation count is not the variable of interest
- The comparison target is a tool other than FMAlign/FMAlign2 with no segment-partitioning output, making segment counts incomparable

## Inputs

- Simulated mitochondrial-like nucleotide sequence datasets at 70-99% similarity (FASTA)
- Human Mitochondrial Genome (mt) dataset FASTA file
- MEM coverage values c = 0.4 and c = 0.8 for the grid sweep
- Minimum MEM length k=39 (parity setting) and -l sweep values around the default 30

## Outputs

- segment_counts.csv with one row per (tool, mode, similarity, c) grid cell plus every -l sweep value
- Non-empty aligned output FASTA files from each FMAlign2 and FMAlign run

## How to apply

Retrieve the simulated mitochondrial-like datasets (70-99% similarity) and the Human Mitochondrial Genome (mt) FASTA files from the Zenodo deposit (zenodo.org/records/10435770). Install FMAlign2 via conda ('conda install -c malab fmalign2'), which bundles MAFFT as the default MSA backend, and run it in global mode with 'FMAlign2 -i <input.fasta> -o <output.fasta>' using minimum MEM length k=39 for parity with FMAlign; repeat in local mode across the c=0.4 and c=0.8 grid cells. Run FMAlign on the same inputs with identical k=39 settings to establish the baseline segment counts, then sweep the -l minimum-MEM-length parameter (default 30) on the mt dataset, lowering it below 30 when too few MEMs are found. Count segments for every run and record tool, mode, similarity, coverage c, -l value, and segment count in segment_counts.csv. Rationale: FMAlign2 redefines FMAlign's global-chain search as partial-chain (suffix-array MEM) search, which markedly augments segmentation, with the local mode generally yielding more segments than the global mode and both converging at similarity extremes.

## Related tools

- **FMAlign2** (Primary aligner run in global and local modes with suffix-array MEM segmentation; -l sets minimum MEM length (default 30), -p selects the MSA backend) — https://github.com/malabz/FMAlign2
- **FMAlign** (Baseline segmenter run with identical k=39 minimum-MEM settings to compare segment counts)
- **MAFFT** (Default MSA backend invoked by FMAlign2 for the sub-MSA of each segment) — https://mafft.cbrc.jp/alignment/software/
- **conda** (Installation channel for FMAlign2 ('conda install -c malab fmalign2') and its MSA dependencies)

## Examples

```
conda install -c malab fmalign2 && FMAlign2 -i input.fasta -o output.fasta -l 39 -p mafft
```

## Evaluation signals

- segment_counts.csv contains one row per (tool, mode, similarity, c) grid cell plus one row per -l sweep value
- Every run produced a non-empty output FASTA file
- FMAlign2's global and local modes produce substantially more segments than FMAlign at k=39 on the simulated datasets and the mt -l sweep
- Local mode segment counts are generally higher than global mode counts at mid-range similarities, with both converging at the 70% and 99% extremes
- Lowering -l below the default 30 increases MEM/segment recovery on the mt dataset when too few MEMs are found

## Limitations

- Both FMAlign2 modes struggle at exceptionally low sequence similarity
- If too few MEMs are found, the default -l=30 must be lowered; if MEM detection takes too long, '-f fast' runs faster but may find fewer MEMs
- Segment counts are tool-specific and only meaningful when the minimum MEM length is matched (k=39) across FMAlign2 and FMAlign

## Evidence

- [abstract] Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy, especially on ultralong datasets.: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy"
- [abstract] FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains.: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains"
- [methods] We recommend installing FMAlign2 with conda.: "We recommend installing **FMAlign2** with **conda**."
- [methods] FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]: "FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]"
- [readme] If too few MEMs are found, try lowering -l (e.g., below 30).: "If too few MEMs are found, try lowering `-l` (e.g., below 30)."
- [abstract] Source code and datasets are available at https://github.com/malabz/FMAlign2 and https://zenodo.org/records/10435770.: "Source code and datasets are available at https://github.com/malabz/FMAlign2 and https://zenodo.org/records/10435770."
