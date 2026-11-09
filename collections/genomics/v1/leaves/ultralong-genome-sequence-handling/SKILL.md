---
name: ultralong-genome-sequence-handling
description: Use when input is a large FASTA of long and similar nucleotide sequences
  (e.g., mtDB benchmark genomes duplicated 20x/50x/100x, or VARV/E.coli-scale datasets)
  where stand-alone MSA tools time out or are prohibitively slow.
when_to_use_negative:
- Sequences are short or highly divergent — FMAlign2's MEM-based segmentation targets
  ultra-long and similar nucleotide sequences; FMAlign's time-saving advantage converges
  with stand-alone methods as sequence count grows (e.g., at mt 100x), so stand-alone
  HAlign3 may be faster on some mtDB inputs
- Alignment involves protein sequences — the method is explicitly a multiple nucleotide
  sequence alignment algorithm
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: FMAlign2
  role: 'Core wrapper: segments ultra-long sequences via suffix-array MEMs, runs subMSAs
    in parallel, and concatenates subsets via sequence-profile alignment and refinement'
  repo: https://github.com/malabz/FMAlign2
- name: MAFFT
  role: Default MSA backend invoked through FMAlign2's -p parameter
- name: Clustal Omega
  role: Alternative MSA backend selectable with -p clustalo
- name: HAlign3
  role: Alternative MSA backend (JAR) selectable with -p halign3; also run stand-alone
    as a baseline (fastest on mtDB among tested methods)
  repo: https://github.com/malabz/HAlign-3/releases/download/v3.0.0-rc1/HAlign-3.0.0_rc1.jar
- name: HAlign4
  role: Alternative MSA backend (C++) selectable with -p halign4
  repo: https://github.com/pinglu-zhang/HAlign-4
- name: sp.py
  role: Python script to compute SP scores of the produced alignments with the documented
    scoring scheme
  repo: https://github.com/malabz/FMAlign2
- name: conda
  role: Recommended installation channel for FMAlign2 and its MSA backend dependencies
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:560a2466fb49f0c3374f6c0cb5b7a84b01897374afb6ca73841575585da0bc9a
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
  - Python
  - conda
  - sp.py
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ultralong-genome-sequence-handling

## Summary

A skill for aligning ultra-long nucleotide sequence sets (sequences approaching billions of bases) by wrapping existing MSA backends (MAFFT, Clustal Omega, HAlign3, HAlign4) with FMAlign2, which segments sequences via suffix-array maximal exact matches (MEMs) and a vertical division strategy with parallel subMSA execution.

## When to use

Use when input is a large FASTA of long and similar nucleotide sequences (e.g., mtDB benchmark genomes duplicated 20x/50x/100x, or VARV/E.coli-scale datasets) where stand-alone MSA tools time out or are prohibitively slow.

## When NOT to use

- Sequences are short or highly divergent — FMAlign2's MEM-based segmentation targets ultra-long and similar nucleotide sequences; FMAlign's time-saving advantage converges with stand-alone methods as sequence count grows (e.g., at mt 100x), so stand-alone HAlign3 may be faster on some mtDB inputs
- Alignment involves protein sequences — the method is explicitly a multiple nucleotide sequence alignment algorithm

## Inputs

- Long and similar nucleotide sequence sets in FASTA format (e.g., VARV, M.genitalium, M.bovis, S.pneumoniae, E.coli datasets from zenodo.org/records/10435770)
- Duplicated mtDB benchmark FASTA files (20x, 50x, 100x duplication levels)
- Optional custom MSA command template file (e.g., mafft-cmd.txt with {input}/{output}/{thread} placeholders)

## Outputs

- Aligned multiple sequence alignment in FASTA format (output.fasta)
- Wall-clock alignment times per method and per duplication level
- SP (Sum-of-Pairs) scores per alignment, e.g., tabulated in mtdb_benchmark_results.csv or sp_runtime_results.tsv

## How to apply

Install FMAlign2 with conda (`conda install -c malab fmalign2`), which by default installs the MAFFT, Clustal Omega, HAlign3, and HAlign4 backends. Run `FMAlign2 -i <input.fasta> -o <output.fasta> -p <mafft|clustalo|halign3|halign4>` on the duplicated FASTA inputs, recording wall-clock times; if MEM detection is slow, switch `-f accurate` to `-f fast` (fewer MEMs found, faster runtime). If too few MEMs are found, lower the minimum MEM length `-l` (default 30). For custom backends, pass a command template file with `{input}`, `{output}`, and optional `{thread}` placeholders via `-p`. Validate quality by computing Sum-of-Pairs scores with sp.py using the documented scheme (match 0, mismatch 1, gap1 2, gap2 2), and verify that FMAlign2 maintains >50% time reduction versus stand-alone baselines even at the largest scale.

## Related tools

- **FMAlign2** (Core wrapper: segments ultra-long sequences via suffix-array MEMs, runs subMSAs in parallel, and concatenates subsets via sequence-profile alignment and refinement) — https://github.com/malabz/FMAlign2
- **MAFFT** (Default MSA backend invoked through FMAlign2's -p parameter)
- **Clustal Omega** (Alternative MSA backend selectable with -p clustalo)
- **HAlign3** (Alternative MSA backend (JAR) selectable with -p halign3; also run stand-alone as a baseline (fastest on mtDB among tested methods)) — https://github.com/malabz/HAlign-3/releases/download/v3.0.0-rc1/HAlign-3.0.0_rc1.jar
- **HAlign4** (Alternative MSA backend (C++) selectable with -p halign4) — https://github.com/pinglu-zhang/HAlign-4
- **sp.py** (Python script to compute SP scores of the produced alignments with the documented scoring scheme) — https://github.com/malabz/FMAlign2
- **conda** (Recommended installation channel for FMAlign2 and its MSA backend dependencies)

## Examples

```
conda install -c malab fmalign2 && FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -p halign3
```

## Evaluation signals

- FMAlign2 runtime at mt (100x) is more than 50% below the stand-alone baseline while SP score is maintained
- All five long-and-similar real datasets (VARV, M.genitalium, M.bovis, S.pneumoniae, E.coli) complete alignment, whereas stand-alone HAlign2 completes none and FMAlign-H2 only three of five
- sp.py returns an SP score for every produced alignment with the documented scheme (match 0, mismatch 1, gap1 2, gap2 2)
- Results table (e.g., mtdb_benchmark_results.csv) contains a row for each tool at each duplication level (20x/50x/100x) with runtime and SP score populated
- HAlign backends resolve correctly: halign3/halign4 available on PATH and FMAlign2 -h lists -p choices mafft|clustalo|halign3|halign4

## Limitations

- If MEM detection takes too long, -f fast speeds things up but may find fewer MEMs (sensitivity trade-off)
- If too few MEMs are found, lower the minimum MEM length -l (default 30)
- FMAlign's time-saving advantage decreases as sequence count grows and converges with stand-alone methods at mt (100x); HAlign3 alone was fastest on mtDB among tested methods
- OpenJDK 11 is only needed if using the HAlign JAR fallback; source builds require g++ (GCC >= 9) and make

## Evidence

- [abstract] FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains.: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [abstract] Vertical division strategy enabling parallel subMSA execution: "By using a vertical division strategy, large-scale problem is deconstructed into manageable tasks, enabling parallel execution of subMSA."
- [methods] Conda installation recommendation and default backends: "We recommend installing **FMAlign2** with **conda**. By default, the Conda environment installs four MSA methods"
- [readme] sp.py evaluation command with scoring scheme: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] -f fast trade-off note: "If MEM detection takes too long, consider `-f fast`; it runs faster but may find fewer MEMs."
