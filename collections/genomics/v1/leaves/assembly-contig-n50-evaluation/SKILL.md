---
name: assembly-contig-n50-evaluation
description: Use when you have raw unscaffolded DBG contigs (e.g., SparseAssembler
  Contigs.txt) and need to compare two parameter runs or check contiguity via N50
  before feeding contigs into a downstream overlap-layout step.
when_to_use_negative:
- The contigs have already been scaffolded, gap-closed, or repeat-resolved — the DBG2OLC
  manual warns such 'improved' contigs introduce heuristic errors and give poor final
  hybrid results
- The goal is final genome assembly quality assessment (e.g., post-consensus backbone),
  not comparison of intermediate DBG contig parameter runs
- Input is long erroneous 3GS reads or a backbone assembly rather than short accurate
  NGS contigs
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST 3.0
  role: Evaluate the assembly results in its default setting and report N50 for each
    SparseAssembler parameter run
- name: SparseAssembler
  role: Produce the raw DBG contigs (Contigs.txt) whose N50 is evaluated; supports
    LD 1 reload to tune NodeCovTh/EdgeCovTh
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Provides AssemblyStatistics to generate two txt files containing essential
    statistics about the assembly
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:c17e0b9a0204f552796adf5fe52659244c4e991d9b69937a69a6420540bfd4a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/SparseAssembler
  tools:
  - AssemblyUtility
  - DBG2OLC
  - QUAST 3.0
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# assembly-contig-n50-evaluation

## Summary

Evaluate de Bruijn graph contig assemblies (e.g., SparseAssembler output Contigs.txt) by computing N50 and related statistics, as in the DBG2OLC hybrid assembly workflow where QUAST 3.0 default settings reported N50 ≈ 29 kbp (default run) and ≈ 32 kbp (tuned run) for 50x Illumina w303 yeast contigs.

## When to use

Use when you have raw unscaffolded DBG contigs (e.g., SparseAssembler Contigs.txt) and need to compare two parameter runs or check contiguity via N50 before feeding contigs into a downstream overlap-layout step.

## When NOT to use

- The contigs have already been scaffolded, gap-closed, or repeat-resolved — the DBG2OLC manual warns such 'improved' contigs introduce heuristic errors and give poor final hybrid results
- The goal is final genome assembly quality assessment (e.g., post-consensus backbone), not comparison of intermediate DBG contig parameter runs
- Input is long erroneous 3GS reads or a backbone assembly rather than short accurate NGS contigs

## Inputs

- Raw unscaffolded de Bruijn graph contigs in FASTA format (SparseAssembler Contigs.txt)
- Illumina short-read FASTQ file (e.g., Illumina_500bp_2x300_R1.fastq.gz, ~50x coverage of S.cer w303)

## Outputs

- QUAST 3.0 assembly evaluation report with N50 (expected ≈29 kbp default run, ≈32 kbp tuned run for the w303 test)
- AssemblyStatistics txt files with essential assembly statistics (optional alternative evaluator)

## How to apply

Run SparseAssembler on ~50x Illumina short reads (e.g., k 51, g 15, NodeCovTh 1, EdgeCovTh 0, GS 12000000 for the S.cer w303 dataset) to produce raw Contigs.txt, then optionally relaunch with LD 1 and tuned NodeCovTh 2 / EdgeCovTh 1 to build a cleaner k-mer graph. Evaluate each contig set with QUAST 3.0 in its default setting and record the reported N50. Accept the run when the N50 matches the expected reference values (≈29 kbp default, ≈32 kbp tuned); alternatively use AssemblyStatistics from the AssemblyUtility repo to generate assembly statistics files. Contiguity must be checked before proceeding: contigs must be raw DBG contigs without repeat-resolving steps such as gap closing or scaffolding, since such heuristics introduce errors that propagate into DBG2OLC's backbone.

## Related tools

- **QUAST 3.0** (Evaluate the assembly results in its default setting and report N50 for each SparseAssembler parameter run)
- **SparseAssembler** (Produce the raw DBG contigs (Contigs.txt) whose N50 is evaluated; supports LD 1 reload to tune NodeCovTh/EdgeCovTh) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Provides AssemblyStatistics to generate two txt files containing essential statistics about the assembly) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f Illumina_50x.fastq && quast.py Contigs.txt
```

## Evaluation signals

- QUAST-reported N50 is approximately 29 kbp for the default SparseAssembler run (LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000)
- QUAST-reported N50 is approximately 32 kbp for the tuned run (LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000)
- Contigs.txt exists and contains raw DBG contigs without scaffolding or gap closing
- Both contig sets (default and tuned) are evaluated under identical QUAST 3.0 default settings so N50 values are comparable

## Limitations

- The beginning of the selected FASTQ can be of lower quality, so the first run may not generate the best result; LD 1 reloading with NodeCovTh 2 / EdgeCovTh 1 may be needed to reach the reported N50 improvement
- N50 is a run-specific reference value ('in my run'); exact N50 may vary between runs, so treat 29/32 kbp as approximate acceptance thresholds
- N50 alone measures contiguity, not correctness; the paper separately checks structural correctness via diagonal dot plots

## Evidence

- [results] We used QUAST 3.032 in its default setting to evaluate the assembly results: "We used QUAST 3.032 in its default setting to evaluate the assembly results"
- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs"
- [readme] In this test run, the N50 is 29 kbp.: "In this test run, the N50 is 29 kbp. As we have selected the beginning part of the sequencing file, which can be of lower quality"
- [readme] The N50 is improved to 32kbp in my run.: "./SparseAssembler LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000 f ../Illumina_data/Illumina_50x.fastq

The N50 is improved to 32kbp in my run."
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
