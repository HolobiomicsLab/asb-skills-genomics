---
name: segment-count-quantification
description: Use when comparing seed-and-partition MSA tools by how many segments
  they generate — e.g. sweeping FMAlign2 vs FMAlign over sequence similarities (70-99%),
  MEM coverage c (0.4-0.8), or minimum MEM length (-l/k=39).
when_to_use_negative:
- The goal is alignment accuracy assessment (SP scores via sp.py) rather than segment-count
  comparison of partitioning behaviour
- Input is a single already-aligned MSA file rather than raw FASTA sequence sets to
  be segmented
- Comparing tools whose partitioning strategy is not seed/MEM-based, where segment
  counts are not comparable to FMAlign/FMAlign2
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: FMAlign2
  role: Primary tool run in global and local modes to produce segments; -l sets minimum
    MEM length (default 30, k=39 for parity with FMAlign)
  repo: https://github.com/malabz/FMAlign2
- name: FMAlign
  role: Baseline segmenter run with identical k=39 settings for segment-count comparison
- name: MAFFT
  role: Default MSA backend used by FMAlign2 to align the produced segments
  repo: https://mafft.cbrc.jp/alignment/software/
- name: conda
  role: Recommended installer for FMAlign2 and its MSA dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:4bb85c65fb9c991e420c426921ac021dc9c7259959533e2429d21f26637cda01
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# segment-count-quantification

## Summary

Quantify the number of segments produced by FMAlign2 (global and local modes) and FMAlign across similarity/coverage grids, the metric used to show FMAlign2 'markedly augments the segmentation of sequences' relative to FMAlign.

## When to use

Use when comparing seed-and-partition MSA tools by how many segments they generate — e.g. sweeping FMAlign2 vs FMAlign over sequence similarities (70-99%), MEM coverage c (0.4-0.8), or minimum MEM length (-l/k=39).

## When NOT to use

- The goal is alignment accuracy assessment (SP scores via sp.py) rather than segment-count comparison of partitioning behaviour
- Input is a single already-aligned MSA file rather than raw FASTA sequence sets to be segmented
- Comparing tools whose partitioning strategy is not seed/MEM-based, where segment counts are not comparable to FMAlign/FMAlign2

## Inputs

- Simulated mitochondrial-like sequence FASTA files (70-99% similarity, MEMs coverage c = 0.4 and c = 0.8)
- Human Mitochondrial Genome (mt) dataset FASTA file
- FMAlign and FMAlign2 run outputs (segmented sequences)

## Outputs

- segment_counts.csv with one row per (tool, mode, similarity, coverage c) grid cell plus every -l sweep value, containing tool, mode, similarity, coverage, -l value, and segment count
- Non-empty aligned/segmented output FASTA per run

## How to apply

Retrieve the simulated mitochondrial-like FASTA datasets and the Human Mitochondrial Genome (mt) dataset from the Zenodo deposit (zenodo.org/records/10435770). Install FMAlign2 via conda and run it in global mode with 'FMAlign2 -i <input.fasta> -o <output.fasta>', setting the minimum MEM length to k=39 (-l 39) for parity with FMAlign, then repeat in local mode across the 70-99% similarity grid at MEM coverage c = 0.4 and c = 0.8. Run FMAlign on the same inputs with identical k=39 settings as the baseline. Sweep the '-l' minimum-MEM-length parameter (default 30) on the mt dataset, lowering it below 30 if too few MEMs are found. Count segments per run and compile tool, mode, similarity, coverage c, and -l value with segment counts into segment_counts.csv. The rationale: FMAlign2's suffix array finds maximal exact matches as partial chains rather than global chains, so its global and local modes produce substantially more segments than FMAlign, with the local mode generally yielding more segments.

## Related tools

- **FMAlign2** (Primary tool run in global and local modes to produce segments; -l sets minimum MEM length (default 30, k=39 for parity with FMAlign)) — https://github.com/malabz/FMAlign2
- **FMAlign** (Baseline segmenter run with identical k=39 settings for segment-count comparison)
- **MAFFT** (Default MSA backend used by FMAlign2 to align the produced segments) — https://mafft.cbrc.jp/alignment/software/
- **conda** (Recommended installer for FMAlign2 and its MSA dependencies)

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 39
```

## Evaluation signals

- segment_counts.csv contains one row per (tool, mode, similarity, c) grid cell plus every -l sweep value
- Every run produced a non-empty output FASTA
- FMAlign2 modes yield substantially more segments than FMAlign at matched k=39 settings
- Local mode generally yields more segments than global mode, with both converging at similarity extremes
- Segment counts change monotonically-sensibly as -l is swept (lowering -l increases MEM hits)

## Limitations

- Both FMAlign2 modes struggle at exceptionally low sequence similarity despite producing more segments
- If too few MEMs are found, -l must be lowered below the default 30, which affects comparability across runs
- FMAlign2's segmentation gain comes from partial chains rather than global chains, so segment counts are not directly a quality measure

## Evidence

- [abstract] FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains.: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [abstract] Compared to FMAlign, FMAlign2 markedly augments segmentation: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy"
- [methods] FMAlign2 usage line: "FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]"
- [readme] -l minimum MEM length: "-l <int>` (default: 30). Minimum MEM length."
- [readme] lower -l when too few MEMs: "If too few MEMs are found, try lowering `-l` (e.g., below 30)."
