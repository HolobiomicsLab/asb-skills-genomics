---
name: de-bruijn-graph-short-read-preassembly
description: Use when starting a DBG2OLC hybrid assembly and you have ~50x Illumina
  FASTQ/FASTQ.gz short reads that must first be converted into raw DBG contigs (Contigs.txt)
  before overlap/layout with long reads.
when_to_use_negative:
- The long-read dataset alone is sufficient and a pure long-read assembler (HGAP,
  Falcon, MHAP-based CA) is intended instead of the hybrid route
- You need finished, scaffolded short-read contigs — scaffolding/gap closing introduces
  errors that the DBG2OLC README warns will produce poor final results
- Coverage is far below ~50x Illumina, where NodeCovTh 1 / EdgeCovTh 0 defaults were
  validated
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SparseAssembler
  role: Sparsified de Bruijn graph assembler producing the raw short-read contigs
    (Contigs.txt); supports LD 1 reload for threshold re-tuning
  repo: https://github.com/yechengxi/SparseAssembler
- name: QUAST 3.0
  role: Evaluates contig sets (default settings) to record N50 for the default vs.
    tuned runs
- name: AssemblyUtility
  role: Provides AssemblyStatistics to evaluate assembly/contig statistics after preassembly
  repo: https://github.com/yechengxi/AssemblyUtility
- name: DBG2OLC
  role: Downstream consumer of Contigs.txt; it compresses long reads onto these short
    accurate contigs via a 17-mer index
  repo: https://github.com/yechengxi/DBG2OLC
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:373a3c55cb28243027815ec13687480ccf8e84f614fada301fb7b591ae18ba0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/SparseAssembler
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# de Bruijn graph short read preassembly

## Summary

Preassemble highly accurate NGS short reads (e.g., 50x Illumina) into raw, unscaffolded de Bruijn graph contigs with SparseAssembler; these contigs become the 'Contigs' input that DBG2OLC compresses long erroneous 3GS reads against in hybrid assembly.

## When to use

Use when starting a DBG2OLC hybrid assembly and you have ~50x Illumina FASTQ/FASTQ.gz short reads that must first be converted into raw DBG contigs (Contigs.txt) before overlap/layout with long reads.

## When NOT to use

- The long-read dataset alone is sufficient and a pure long-read assembler (HGAP, Falcon, MHAP-based CA) is intended instead of the hybrid route
- You need finished, scaffolded short-read contigs — scaffolding/gap closing introduces errors that the DBG2OLC README warns will produce poor final results
- Coverage is far below ~50x Illumina, where NodeCovTh 1 / EdgeCovTh 0 defaults were validated

## Inputs

- Highly accurate NGS short reads in FASTA/FASTQ format (e.g., Illumina_500bp_2x300_R1.fastq.gz, ~50x coverage)
- Genome size estimate in bp (GS parameter, suggest ~3x genome size)
- Previous SparseAssembler saved k-mer graph (for LD 1 reload/re-tuning)

## Outputs

- Contigs.txt: raw, unscaffolded de Bruijn graph contigs in FASTA format, consumed by DBG2OLC as the NGS_CONTIG_FILE
- QUAST 3.0 / AssemblyStatistics reports (N50 and assembly statistics) for the contig set

## How to apply

Compile SparseAssembler from github.com/yechengxi/SparseAssembler with 'g++ -O3 -o SparseAssembler *.cpp', then run it with a sparsified de Bruijn graph parameterization (k 51, g 15, NodeCovTh 1, EdgeCovTh 0, GS set to ~genome size or larger, e.g., GS 12000000 for the 12 Mbp S.cer w303 yeast dataset) on the Illumina FASTQ file, producing Contigs.txt. Critically, the contigs must be raw DBG contigs — do not apply repeat-resolving steps such as gap closing or scaffolding, since the heuristics in short-read pipelines introduce errors that degrade the final hybrid assembly. If the first run yields a poor graph (e.g., N50 near 29 kbp on the w303 test data), reload the saved k-mer graph with LD 1 and tune the spurious-node/link thresholds (e.g., NodeCovTh 2, EdgeCovTh 1), which improved the test N50 to ~32 kbp. Evaluate contig quality with QUAST 3.0 in default settings or AssemblyStatistics before feeding Contigs.txt to the DBG2OLC overlap/layout step.

## Related tools

- **SparseAssembler** (Sparsified de Bruijn graph assembler producing the raw short-read contigs (Contigs.txt); supports LD 1 reload for threshold re-tuning) — https://github.com/yechengxi/SparseAssembler
- **QUAST 3.0** (Evaluates contig sets (default settings) to record N50 for the default vs. tuned runs)
- **AssemblyUtility** (Provides AssemblyStatistics to evaluate assembly/contig statistics after preassembly) — https://github.com/yechengxi/AssemblyUtility
- **DBG2OLC** (Downstream consumer of Contigs.txt; it compresses long reads onto these short accurate contigs via a 17-mer index) — https://github.com/yechengxi/DBG2OLC

## Examples

```
./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f Illumina_50x.fastq
```

## Evaluation signals

- Contigs.txt exists and contains raw (non-scaffolded) FASTA contigs from the DBG run
- QUAST-reported N50 ≈ 29 kbp for the default w303 run (LD 0, k 51, g 15, NodeCovTh 1, EdgeCovTh 0) and ≈ 32 kbp after LD 1 retuning with NodeCovTh 2, EdgeCovTh 1
- Contigs lack scaffolding/gap-closing artifacts — contig IDs are plain DBG contig names (>Contig_xxx style) usable as anchors
- Downstream DBG2OLC run accepts Contigs.txt and produces a backbone whose N50 satisfies the pre-consensus checkpoint before Sparc polishing

## Limitations

- First-run parameters (NodeCovTh 1, EdgeCovTh 0) may not suit complex genomes or different coverages; LD 1 reload with NodeCovTh/EdgeCovTh retuning may be required
- Using contigs processed by repeat-resolving heuristics (gap closing, scaffolding) degrades final hybrid assembly quality because those steps introduce errors
- Beginning-of-file read subsets may be lower quality, depressing N50 (as in the w303 test run at 29 kbp vs. 32 kbp after retuning)

## Evidence

- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
- [readme] ./SparseAssembler LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000 f ../Illumina_data/Illumina_50x.fastq: "./SparseAssembler LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000 f ../Illumina_data/Illumina_50x.fastq"
- [results] We used QUAST 3.032 in its default setting to evaluate the assembly results: "We used QUAST 3.032 in its default setting to evaluate the assembly results"
