---
name: msa-runtime-benchmarking
description: Use when comparing MSA tools on ultralong (>=100,000 bp up to multi-Mbp)
  nucleotide sequences and you need success/failure outcomes plus wall-clock runtime
  per method and per sequence length.
when_to_use_negative:
- Sequences are short (well under 100,000 bp) or few in number — standard MSA tools
  handle these directly without the ultralong benchmarking harness
- Input molecules are proteins — FMAlign2 is a multiple *nucleotide* sequence alignment
  method
- The goal is accuracy benchmarking on reference alignments rather than success/failure
  and runtime scaling on ultralong inputs
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: FMAlign2
  role: Wraps MSA backends and performs suffix-array MEM-based segmentation to enable
    alignment of ultralong sequences; invoked with -i/-o/-p/-t/-l/-f for the benchmark
  repo: https://github.com/malabz/FMAlign2
- name: MAFFT
  role: Default MSA backend run through FMAlign2 (-p mafft) or a custom command file
    with extra parameters
- name: Clustal Omega
  role: MSA backend selectable with -p clustalo
  repo: http://www.clustal.org/omega/
- name: HAlign3
  role: MSA backend selectable with -p halign3; fails on sequences longer than ~100,000
    bp in the benchmark
  repo: https://github.com/malabz/HAlign-3
- name: HAlign4
  role: MSA backend selectable with -p halign4
  repo: https://github.com/pinglu-zhang/HAlign-4
- name: FMAlign
  role: Predecessor method that handles million-base sequences but is outperformed
    in runtime by FMAlign2 on 10 Mbp+ inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:4be72cff8ea4ba2da74521b53c27865115cd2677fd0f89be59fd01a26b52da4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - Clustal Omega
  - FMAlign
  - FMAlign2
  - HAlign3
  - HAlign4
  - MAFFT
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# msa-runtime-benchmarking

## Summary

Benchmark multiple sequence alignment methods on ultralong nucleotide datasets by timing success/failure across truncated human Y chromosome inputs (10,000 bp to 15 Mbp), with FMAlign2 wrapping backends like MAFFT, Clustal Omega, HAlign3, and HAlign4.

## When to use

Use when comparing MSA tools on ultralong (>=100,000 bp up to multi-Mbp) nucleotide sequences and you need success/failure outcomes plus wall-clock runtime per method and per sequence length.

## When NOT to use

- Sequences are short (well under 100,000 bp) or few in number — standard MSA tools handle these directly without the ultralong benchmarking harness
- Input molecules are proteins — FMAlign2 is a multiple *nucleotide* sequence alignment method
- The goal is accuracy benchmarking on reference alignments rather than success/failure and runtime scaling on ultralong inputs

## Inputs

- Truncated human Y chromosome FASTA files (10,000 bp to 15 Mbp) from Zenodo record 10435770
- Installed FMAlign2 with conda-bundled MSA backends (MAFFT, Clustal Omega, HAlign3, HAlign4)
- Optional custom MSA command file (e.g. mafft-cmd.txt with {thread}/{input}/{output} placeholders)

## Outputs

- Benchmark table of run success/failure and wall-clock runtime per method per truncation length
- Aligned output FASTA files per method/length combination
- SP (Sum-of-Pairs) scores computed with sp.py (match 0, mismatch 1, gap1 2, gap2 2)

## How to apply

Install FMAlign2 and its bundled backends via conda (`conda install -c malab fmalign2`, which ships MAFFT, Clustal Omega, HAlign3, HAlign4). Obtain the truncated human Y chromosome FASTA datasets (10,000 bp to 15 Mbp) from the Zenodo deposit. For each truncation length and backend, run `FMAlign2 -i input.fasta -o output.fasta -p <method> -t <threads>`, fixing the thread count (`-t`) so runtime comparisons are fair; supply a custom command file (e.g. `mafft --retree 1 --thread {thread} {input} > {output}`) via `-p /path/to/mafft-cmd.txt` when extra backend parameters are needed. Keep defaults `-l 30` and `-f accurate` unless a run stalls, in which case retry with `-f fast`; record whether each run completed and its wall-clock runtime. Verify completed alignments by scoring with the sp.py Sum-of-Pairs script using match 0, mismatch 1, gap1 2, gap2 2, then compile success/failure and runtime into a single benchmark table. The expected pattern is that HAlign2-centered methods fail beyond ~100,000 bp while FMAlign2 returns results within an acceptable timeframe even for sequences 10 Mbp or longer.

## Related tools

- **FMAlign2** (Wraps MSA backends and performs suffix-array MEM-based segmentation to enable alignment of ultralong sequences; invoked with -i/-o/-p/-t/-l/-f for the benchmark) — https://github.com/malabz/FMAlign2
- **MAFFT** (Default MSA backend run through FMAlign2 (-p mafft) or a custom command file with extra parameters)
- **Clustal Omega** (MSA backend selectable with -p clustalo) — http://www.clustal.org/omega/
- **HAlign3** (MSA backend selectable with -p halign3; fails on sequences longer than ~100,000 bp in the benchmark) — https://github.com/malabz/HAlign-3
- **HAlign4** (MSA backend selectable with -p halign4) — https://github.com/pinglu-zhang/HAlign-4
- **FMAlign** (Predecessor method that handles million-base sequences but is outperformed in runtime by FMAlign2 on 10 Mbp+ inputs)

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -p mafft -t 8 && python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2
```

## Evaluation signals

- Every run's completion status and wall-clock runtime are recorded for each (method, truncation length) pair
- Completed alignments yield an SP score via sp.py with scoring --match 0 --mismatch 1 --gap1 2 --gap2 2
- Reproduces the expected pattern: HAlign2-centered methods fail past 100,000 bp while FMAlign2 completes 10 Mbp+ inputs in acceptable time
- Thread count (-t) is fixed across all compared methods so timings are fair
- Default parameters (-l 30, -f accurate) are used unless a stalled run is retried with -f fast, and retries are noted

## Limitations

- If too few MEMs are found, -l may need to be lowered below 30; if MEM detection takes too long, -f fast runs faster but finds fewer MEMs at some cost to sensitivity
- HAlign2-centered methods fail once sequence length surpasses 100,000 bp, limiting comparability at the long end of the range
- The OpenJDK 11 runtime is only needed for the HAlign JAR fallback, an extra dependency for some backends
- The -t default (all available cores) must be overridden to a fixed value or runtime comparisons across methods are unfair

## Evidence

- [supplementary] HAlign2-centered methods fail once sequence length surpasses 100,000 bp, while FMAlign2 succeeds on ultralong sequences: [evidence span withheld — anchor status: not_found]
- [readme] FMAlign2 command with backend selection: "FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta [other options]"
- [methods] Bundled MSA backends via conda: "We recommend installing **FMAlign2** with **conda**.
By default, the Conda environment installs four MSA methods"
- [readme] SP score evaluation command: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] Custom MSA command file format: "mafft --retree 1 --thread {thread} {input} > {output}"
