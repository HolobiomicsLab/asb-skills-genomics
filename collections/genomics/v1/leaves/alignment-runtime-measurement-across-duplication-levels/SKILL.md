---
name: alignment-runtime-measurement-across-duplication-levels
description: Use when comparing wall-clock alignment time and SP accuracy of MSA tools
  as sequence count grows multiplicatively (20x/50x/100x duplication), especially
  to test whether an accelerator like FMAlign2 retains >50% time savings at the largest
  scale.
when_to_use_negative:
- Sequences are ultra-long but not numerous — the benchmark targets scaling in sequence
  count (duplication levels), not per-sequence length variation
- You need alignment accuracy alone without runtime measurement — a single SP-score
  evaluation suffices rather than the full timing matrix
- Protein sequences — FMAlign2 is designed for nucleotide sequence alignment
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: FMAlign2
  role: Accelerator wrapper whose per-backend runtime is measured across duplication
    levels
  repo: https://github.com/malabz/FMAlign2
- name: MAFFT
  role: MSA backend run both through FMAlign2 (-p mafft) and standalone as a timing
    baseline
  repo: https://mafft.cbrc.jp/alignment/software/
- name: Clustal Omega
  role: MSA backend run both through FMAlign2 (-p clustalo) and standalone as a timing
    baseline
  repo: http://www.clustal.org/omega/
- name: HAlign3
  role: MSA backend run through FMAlign2 (-p halign3) and standalone; fastest standalone
    method on mtDB among tested
  repo: https://github.com/malabz/HAlign-3
- name: HAlign4
  role: MSA backend run through FMAlign2 (-p halign4) and standalone as a timing baseline
  repo: https://github.com/pinglu-zhang/HAlign-4
- name: conda
  role: Installation channel for FMAlign2 and its default MSA backends
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:be77468fb910b1dcb580f855f61e6025115ed323c40459cc8aee842bda32f1d8
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
  - conda
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# alignment-runtime-measurement-across-duplication-levels

## Summary

Reproduces the mtDB large-scale acceleration benchmark (Suppl. Table S2) by timing FMAlign2 and four MSA backends (MAFFT, Clustal Omega, HAlign3, HAlign4) on mitochondrial genomes duplicated 20x, 50x, and 100x, and verifying with SP scores that speedups do not sacrifice alignment accuracy.

## When to use

Use when comparing wall-clock alignment time and SP accuracy of MSA tools as sequence count grows multiplicatively (20x/50x/100x duplication), especially to test whether an accelerator like FMAlign2 retains >50% time savings at the largest scale.

## When NOT to use

- Sequences are ultra-long but not numerous — the benchmark targets scaling in sequence count (duplication levels), not per-sequence length variation
- You need alignment accuracy alone without runtime measurement — a single SP-score evaluation suffices rather than the full timing matrix
- Protein sequences — FMAlign2 is designed for nucleotide sequence alignment

## Inputs

- Duplicated mtDB mitochondrial genome FASTA files at 20x, 50x, and 100x duplication levels (from https://zenodo.org/records/10435770)
- Installed MSA tools: FMAlign2, MAFFT, Clustal Omega, HAlign3, HAlign4 (and FMAlign for comparison)

## Outputs

- Per-method wall-clock alignment times and SP scores across 20x/50x/100x duplication levels, written to mtdb_benchmark_results.csv
- Aligned FASTA outputs from FMAlign2 and each standalone baseline method

## How to apply

Install FMAlign2 via conda (conda install -c malab fmalign2), which by default installs the MAFFT, Clustal Omega, HAlign3, and HAlign4 backends. For each duplicated mtDB FASTA (20x, 50x, 100x), run FMAlign2 with each backend using '-p mafft|clustalo|halign3|halign4' and record wall-clock time; switch to '-f fast' only if MEM detection is slow, accepting the documented sensitivity trade-off. Run the standalone MSA tools (and FMAlign itself) directly on the same inputs as baselines. Compute SP scores for every output alignment with sp.py (match 0, mismatch 1, gap1 2, gap2 2) to confirm accuracy is maintained. Tabulate per-method times and SP scores across duplication levels and check that FMAlign2 at mt (100x) is more than 50% below baseline runtime, writing results to mtdb_benchmark_results.csv.

## Related tools

- **FMAlign2** (Accelerator wrapper whose per-backend runtime is measured across duplication levels) — https://github.com/malabz/FMAlign2
- **MAFFT** (MSA backend run both through FMAlign2 (-p mafft) and standalone as a timing baseline) — https://mafft.cbrc.jp/alignment/software/
- **Clustal Omega** (MSA backend run both through FMAlign2 (-p clustalo) and standalone as a timing baseline) — http://www.clustal.org/omega/
- **HAlign3** (MSA backend run through FMAlign2 (-p halign3) and standalone; fastest standalone method on mtDB among tested) — https://github.com/malabz/HAlign-3
- **HAlign4** (MSA backend run through FMAlign2 (-p halign4) and standalone as a timing baseline) — https://github.com/pinglu-zhang/HAlign-4
- **conda** (Installation channel for FMAlign2 and its default MSA backends)

## Examples

```
FMAlign2 -i mt_100x.fasta -o mt_100x_f2_mafft.fasta -p mafft; python sp.py --input mt_100x_f2_mafft.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2
```

## Evaluation signals

- FMAlign2 maintains over 50% time reduction versus the standalone baseline even at mt (100x)
- SP scores computed with sp.py (--match 0 --mismatch 1 --gap1 2 --gap2 2) confirm FMAlign2 maintains accuracy relative to standalone methods
- Timing matrix covers all 4 backends × 3 duplication levels plus standalone baselines, including FMAlign for comparison
- Observed trend reproduces the paper's finding: FMAlign's time-saving advantage decreases with sequence count, converging with stand-alone methods at mt (100x), while FMAlign2 does not
- HAlign3 standalone remains the fastest method on mtDB among those tested

## Limitations

- FMAlign's advantage converges with stand-alone methods at mt (100x) — FMAlign, not FMAlign2, loses its speed edge at the largest duplication level
- The '-f fast' MEM filtering mode runs faster but may find fewer MEMs, at the cost of sensitivity
- If too few MEMs are found, the minimum MEM length (-l, default 30) may need lowering below 30

## Evidence

- [supplementary] FMAlign shows a decreasing time-saving advantage as sequence count grows, converging with stand-alone methods at mt (100x), whereas FMAlign2 maintains over 50% time reduction even at mt (100x): [evidence span withheld — anchor status: not_found]
- [readme] Conda install of FMAlign2 with four backends: "By default, the Conda environment installs four MSA methods: [MAFFT](https://mafft.cbrc.jp/alignment/software/), [Clustal Omega](http://www.clustal.org/omega/), HAlign3, and [HAlign4]"
- [readme] SP score evaluation command: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] Fast mode trade-off: "`-f <mode>` (default: `accurate`). MEM filtering mode; use `fast` to speed up at the cost of sensitivity."
- [abstract] FMAlign2 compared to FMAlign reduces time while maintaining accuracy: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy"
