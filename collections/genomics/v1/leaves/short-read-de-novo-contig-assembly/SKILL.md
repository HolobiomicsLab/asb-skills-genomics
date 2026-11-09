---
name: short-read-de-novo-contig-assembly
description: Use when ~50x Illumina FASTQ reads are available and accurate short contigs
  are needed as the contig input for a DBG2OLC hybrid assembly or similar long-read
  scaffolding pipeline.
when_to_use_negative:
- When the goal is a standalone finished short-read assembly — these raw contigs deliberately
  skip gap closing and scaffolding and are meant only as input to long-read hybrid
  assembly
- When only long reads (PacBio/ONT) are available and no Illumina contig file can
  be produced
- When a previously scaffolded or gap-closed short-read assembly is all that exists
  — the README warns this 'may have poor final results due to the errors introduced
  by the heuristics'
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SparseAssembler
  role: Compact de Bruijn graph assembler used to preassemble 50x Illumina reads into
    raw DBG contigs (Contigs.txt)
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Provides AssemblyStatistics to evaluate contig N50 and other assembly statistics
  repo: https://github.com/yechengxi/AssemblyUtility
- name: DBG2OLC
  role: Downstream consumer of the Contigs.txt, which compresses long reads onto these
    contigs during overlap and layout
  repo: https://github.com/yechengxi/DBG2OLC
- name: QUAST 3.0
  role: Evaluates the final assembly results in default setting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:d0a1b685b4054eb843e35a4ad8736c04c91263253cc6df667fd1aaf1d89dfa42
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# short-read-de-novo-contig-assembly

## Summary

Preassemble ~50x Illumina short reads into short but accurate raw de Bruijn graph (DBG) contigs with SparseAssembler, e.g. k 51, g 15, NodeCovTh 1, EdgeCovTh 0, GS 12000000 on the S.cer w303 dataset, to serve as high-quality anchors for long-read hybrid assembly.

## When to use

Use when ~50x Illumina FASTQ reads are available and accurate short contigs are needed as the contig input for a DBG2OLC hybrid assembly or similar long-read scaffolding pipeline.

## When NOT to use

- When the goal is a standalone finished short-read assembly — these raw contigs deliberately skip gap closing and scaffolding and are meant only as input to long-read hybrid assembly
- When only long reads (PacBio/ONT) are available and no Illumina contig file can be produced
- When a previously scaffolded or gap-closed short-read assembly is all that exists — the README warns this 'may have poor final results due to the errors introduced by the heuristics'

## Inputs

- ~50x Illumina paired-end FASTQ reads (e.g. Illumina_500bp_2x300_R1.fastq from the S.cer w303 dataset)
- Estimated genome size in bp (GS, e.g. 12000000 for yeast)

## Outputs

- Raw DBG contigs in FASTA format (SparseAssembler Contigs.txt)
- Contig assembly statistics (e.g. contig N50 from AssemblyStatistics/QUAST)

## How to apply

Run SparseAssembler on the Illumina FASTQ with GS set to an estimate of genome size (12,000,000 bp for the 12 Mbp yeast w303 genome), k 51, g 15, and at ~50x coverage use NodeCovTh 1 EdgeCovTh 0, e.g. './SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f Illumina_50x.fastq'. Critically, the output Contigs.txt must be raw DBG contigs produced WITHOUT repeat-resolving heuristics such as gap closing or scaffolding, because those heuristics introduce errors that degrade the final hybrid assembly. If the first run is unsatisfactory (the w303 test gave N50 29 kbp, improvable to 32 kbp), reload the saved k-mer graph with LD 1 and retune NodeCovTh (e.g. 2) and EdgeCovTh (e.g. 1). Feed the resulting Contigs.txt (FASTA) to the downstream overlap-layout step (DBG2OLC) and assess contiguity with AssemblyStatistics or QUAST.

## Related tools

- **SparseAssembler** (Compact de Bruijn graph assembler used to preassemble 50x Illumina reads into raw DBG contigs (Contigs.txt)) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Provides AssemblyStatistics to evaluate contig N50 and other assembly statistics) — https://github.com/yechengxi/AssemblyUtility
- **DBG2OLC** (Downstream consumer of the Contigs.txt, which compresses long reads onto these contigs during overlap and layout) — https://github.com/yechengxi/DBG2OLC
- **QUAST 3.0** (Evaluates the final assembly results in default setting)

## Examples

```
./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f Illumina_50x.fastq
```

## Evaluation signals

- w303 test run at NodeCovTh 1 EdgeCovTh 0 gives contig N50 ~29 kbp; retuning with LD 1 NodeCovTh 2 EdgeCovTh 1 improved N50 to ~32 kbp
- Output file is named Contigs.txt and is a FASTA of raw DBG contigs with no scaffolding/gap-closing applied
- Downstream DBG2OLC run on 20x PacBio reads achieves backbone N50 583 kbp, confirming contig quality
- GS parameter matches genome size estimate (12 Mbp for w303) for correct memory pre-allocation
- Final polished assembly N50 evaluated with AssemblyStatistics or QUAST 3.0 in default setting

## Limitations

- Contigs must be raw DBG contigs; using repeat-resolving techniques such as gap closing or scaffolding introduces errors and can yield poor final hybrid results
- The beginning of a sequencing file can be of lower quality, degrading the first assembly (the w303 test selected the beginning of the file)
- For other genomes or coverages the first parameterization may not be optimal; NodeCovTh/EdgeCovTh may need retuning via LD 1 reload

## Evidence

- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding. Otherwise you may have poor final results"
- [readme] SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000: "./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f ../Illumina_data/Illumina_50x.fastq"
- [readme] Normally with ~50x coverage, NodeCovTh 1 EdgeCovTh 0 can produce nice results.: "Normally with ~50x coverage, NodeCovTh 1 EdgeCovTh 0 can produce nice results."
- [readme] The previous computations can be loaded and two parameters can be fine-tuned to construct a cleaner de Bruijn/ k-mer graph: "The previous computations can be loaded and two parameters can be fine-tuned to construct a cleaner de Bruijn/ k-mer graph"
