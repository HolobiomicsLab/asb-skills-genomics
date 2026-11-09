---
name: runtime-memory-profiling
description: Use when comparing MSA tool variants on nucleotide FASTA inputs and you
  need per-run wall-clock time, peak memory, and Q/TC-quality evidence showing how
  performance changes with sequence similarity.
when_to_use_negative:
- The input is not nucleotide FASTA sequence data or the goal is protein MSA benchmarking
- You need absolute accuracy against a reference alignment rather than runtime/memory
  scaling behavior — this skill centers on profiling, with SP scores only as a sanity
  check that accuracy is maintained
- Only a single configuration is of interest with no comparison across backends or
  similarity levels
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0102
tools:
- name: FMAlign2
  role: Tool under benchmark; invoked with -i/-o/-p to produce alignments whose runtime
    and memory are profiled
  repo: https://github.com/malabz/FMAlign2
- name: MAFFT
  role: Default MSA backend for FMAlign2 (FMAlign2-M variant)
- name: HAlign3
  role: Alternative MSA backend for FMAlign2 (FMAlign2-H3 variant), installed as JAR
    requiring OpenJDK 11
  repo: https://github.com/malabz/HAlign-3/releases/download/v3.0.0-rc1/HAlign-3.0.0_rc1.jar
- name: HAlign4
  role: Additional MSA backend installable via conda for FMAlign2 (-p halign4)
  repo: https://github.com/pinglu-zhang/HAlign-4
- name: sp.py
  role: Evaluation script computing Sum-of-Pairs scores for each output alignment
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:fc52ee519d9fc45feeea463f95a2774c25de1fc55a5a86017ccd2aeb354835df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - Clustal Omega
  - FMAlign2
  - HAlign3
  - HAlign4
  - MAFFT
  - OpenJDK 11
  - sp.py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# runtime-memory-profiling

## Summary

Benchmark how runtime, peak memory, and alignment accuracy of FMAlign2 (with MAFFT, HAlign2, or HAlign3 backends) scale with sequence similarity on FASTA datasets, wrapping each run with wall-clock and peak-memory measurement and scoring outputs with sp.py SP scores.

## When to use

Use when comparing MSA tool variants on nucleotide FASTA inputs and you need per-run wall-clock time, peak memory, and Q/TC-quality evidence showing how performance changes with sequence similarity.

## When NOT to use

- The input is not nucleotide FASTA sequence data or the goal is protein MSA benchmarking
- You need absolute accuracy against a reference alignment rather than runtime/memory scaling behavior — this skill centers on profiling, with SP scores only as a sanity check that accuracy is maintained
- Only a single configuration is of interest with no comparison across backends or similarity levels

## Inputs

- Nucleotide FASTA alignment input (e.g. INDELible-simulated mitochondrial-like dataset of 100 sequences at 90–99% similarity, Zenodo record 10435770)
- FMAlign2 backend choice (-p mafft | clustalo | halign3 | halign4, or a custom MSA command file)

## Outputs

- Aligned FASTA file per FMAlign2 variant
- Per-run wall-clock time and peak memory (e.g. from /usr/bin/time -v)
- Sum-of-Pairs (SP) scores and derived Q/TC scores per alignment
- Compiled benchmark table (fmalign2_benchmark_results.csv)

## How to apply

Install FMAlign2 via conda ('conda install -c malab fmalign2'), which brings in the MSA backends (MAFFT, Clustal Omega, HAlign3, HAlign4; OpenJDK 11 is needed for the HAlign JAR fallback). Run each backend variant with 'FMAlign2 -i input.fasta -o output.fasta -p <method>' keeping default settings (minimum MEM length -l 30, accurate MEM filtering mode -f accurate), and wrap every run with wall-clock and peak-memory measurement such as /usr/bin/time -v. Evaluate each output alignment with the sp.py script using the scoring scheme --match 0 --mismatch 1 --gap1 2 --gap2 2 to obtain Sum-of-Pairs scores, from which Q/TC scores are derived. Compile per-variant runtime, peak-memory, and Q/TC values into a single table (e.g. fmalign2_benchmark_results.csv); on the simulated mitochondrial-like 100-sequence dataset (90–99% similarity), higher similarity should yield lower time and memory across FMAlign2-M, FMAlign2-H2, and FMAlign2-H3.

## Related tools

- **FMAlign2** (Tool under benchmark; invoked with -i/-o/-p to produce alignments whose runtime and memory are profiled) — https://github.com/malabz/FMAlign2
- **MAFFT** (Default MSA backend for FMAlign2 (FMAlign2-M variant))
- **HAlign3** (Alternative MSA backend for FMAlign2 (FMAlign2-H3 variant), installed as JAR requiring OpenJDK 11) — https://github.com/malabz/HAlign-3/releases/download/v3.0.0-rc1/HAlign-3.0.0_rc1.jar
- **HAlign4** (Additional MSA backend installable via conda for FMAlign2 (-p halign4)) — https://github.com/pinglu-zhang/HAlign-4
- **sp.py** (Evaluation script computing Sum-of-Pairs scores for each output alignment) — https://github.com/malabz/FMAlign2

## Examples

```
FMAlign2 -i input.fasta -o output.fasta -p mafft && python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2
```

## Evaluation signals

- Each variant's output alignment exists and is valid FASTA
- sp.py produces a computable SP score for every output alignment
- Runtime and peak memory decline monotonically as input sequence similarity rises from 90% to 99% for FMAlign2-M, FMAlign2-H2, and FMAlign2-H3
- Accuracy (Q/TC derived from SP scores) is maintained while runtime is reduced relative to plain MSA backends, reproducing the reported FMAlign2 behavior
- Default FMAlign2 settings were used: -l 30 (minimum MEM length) and -f accurate (MEM filtering mode)

## Limitations

- With too few MEMs found, lower -l (e.g. below 30); if MEM detection is too slow, -f fast runs faster but may find fewer MEMs at a sensitivity cost
- The HAlign JAR fallback requires OpenJDK 11 specifically
- Reported trends are on simulated mitochondrial-like data (100 sequences, 90–99% similarity); absolute timings/memories will differ on other hardware and datasets

## Evidence

- [methods] usage command: "FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]"
- [abstract] accuracy claim: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy, especially on ultralong datasets."
- [readme] evaluation command: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] parameter tuning note: "If too few MEMs are found, try lowering `-l` (e.g., below 30)."
- [readme] backend note: "OpenJDK 11 (only needed if you use the HAlign JAR fallback)"
