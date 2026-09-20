---
name: sparse-assembler-parameter-tuning
description: Use when you have ~50x Illumina FASTQ reads of a small genome and need
  raw, unscaffolded DBG contigs for the DBG2OLC Step1, and the first SparseAssembler
  run's N50 is unsatisfactory.
when_to_use_negative:
- You need finished, scaffolded assemblies — SparseAssembler output must be raw DBG
  contigs without gap closing or scaffolding, which otherwise introduces errors into
  the DBG2OLC pipeline.
- You are assembling long reads only (non-hybrid); this tuning targets the NGS contig
  construction step of the DBG2OLC hybrid workflow.
- Coverage is far from ~50x — the NodeCovTh 1 / EdgeCovTh 0 defaults are stated for
  ~50x coverage and other coverages require re-tuning.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SparseAssembler
  role: Constructs raw de Bruijn graph contigs from Illumina short reads; LD 1 reloads
    the saved k-mer graph so NodeCovTh and EdgeCovTh can be fine-tuned
  repo: https://github.com/yechengxi/SparseAssembler
- name: QUAST 3.0
  role: Evaluates the contig sets (N50) from both the default and tuned runs
- name: DBG2OLC
  role: Consumes the resulting Contigs.txt in Step2 (Overlap and layout) with k 17
    to anchor contigs to long reads
  repo: https://github.com/yechengxi/DBG2OLC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:6324136d683722577ff182aa6f022d8d36d5e327f7d04c53bde30a7013029ecc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/SparseAssembler
  tools:
  - DBG2OLC
  - QUAST 3.0
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sparse-assembler-parameter-tuning

## Summary

Run SparseAssembler twice on ~50x Illumina short reads to produce raw de Bruijn graph contigs, then reload the saved k-mer graph (LD 1) and tune NodeCovTh/EdgeCovTh to build a cleaner graph, improving the assembly N50 (29 kbp default → 32 kbp tuned on the S.cer w303 dataset).

## When to use

Use when you have ~50x Illumina FASTQ reads of a small genome and need raw, unscaffolded DBG contigs for the DBG2OLC Step1, and the first SparseAssembler run's N50 is unsatisfactory.

## When NOT to use

- You need finished, scaffolded assemblies — SparseAssembler output must be raw DBG contigs without gap closing or scaffolding, which otherwise introduces errors into the DBG2OLC pipeline.
- You are assembling long reads only (non-hybrid); this tuning targets the NGS contig construction step of the DBG2OLC hybrid workflow.
- Coverage is far from ~50x — the NodeCovTh 1 / EdgeCovTh 0 defaults are stated for ~50x coverage and other coverages require re-tuning.

## Inputs

- ~50x Illumina short-read FASTQ file (e.g. Illumina_500bp_2x300_R1.fastq.gz from the S.cer w303 dataset)
- Genome size estimate in bp (GS 12000000 for the 12 Mbp yeast genome)

## Outputs

- Contigs.txt — raw, unscaffolded de Bruijn graph contigs in fasta format, used by DBG2OLC Step2
- QUAST 3.0 assembly statistics, especially N50 (~29 kbp default run, ~32 kbp tuned run)

## How to apply

Download the Illumina FASTQ (e.g. Illumina_500bp_2x300_R1.fastq.gz from the S.cer w303 dataset) and run SparseAssembler with 'LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000' — NodeCovTh 1 and EdgeCovTh 0 are recommended for ~50x coverage and yield an N50 of ~29 kbp in the paper's test run. Evaluate with QUAST 3.0 in default setting. If quality is poor (partly because the beginning of the sequencing file can be lower quality), relaunch SparseAssembler with 'LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000' to reload the previous computation and fine-tune the two false k-mer/edge thresholds for a cleaner de Bruijn graph; the paper's tuned run reached N50 ~32 kbp. Contigs must be raw DBG contigs without repeat resolving (no gap closing or scaffolding), because scaffolding heuristics introduce errors that degrade the downstream DBG2OLC hybrid assembly. The output Contigs.txt is then fed to DBG2OLC in Step2 (k 17 anchors contigs to PacBio long reads).

## Related tools

- **SparseAssembler** (Constructs raw de Bruijn graph contigs from Illumina short reads; LD 1 reloads the saved k-mer graph so NodeCovTh and EdgeCovTh can be fine-tuned) — https://github.com/yechengxi/SparseAssembler
- **QUAST 3.0** (Evaluates the contig sets (N50) from both the default and tuned runs)
- **DBG2OLC** (Consumes the resulting Contigs.txt in Step2 (Overlap and layout) with k 17 to anchor contigs to long reads) — https://github.com/yechengxi/DBG2OLC

## Examples

```
./SparseAssembler LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000 f ../Illumina_data/Illumina_50x.fastq
```

## Evaluation signals

- Contigs.txt exists and is in fasta format (raw DBG contigs, no scaffolding).
- QUAST 3.0 default-setting N50 is approximately 29 kbp for the LD 0 / NodeCovTh 1 / EdgeCovTh 0 run on the w303 dataset.
- The tuned LD 1 run (NodeCovTh 2, EdgeCovTh 1) improves N50 to approximately 32 kbp.
- NodeCovTh and EdgeCovTh remain within their supported ranges (0–16), with NodeCovTh 1 / EdgeCovTh 0 as the ~50x coverage defaults.
- Contigs are accepted downstream by './DBG2OLC k 17 ... Contigs Contigs.txt f Pacbio_20x.fasta' in Step2 without errors.

## Limitations

- For genomes other than the simple yeast test case, or different coverage, the first run may not produce the best result and requires the LD 1 reload-and-tune cycle.
- Using scaffolding/gap-closing contigs instead of raw DBG contigs causes poor final results due to errors introduced by short-read assembly heuristics.
- The beginning of the sequencing file can be of lower quality, lowering the default-run N50 (29 kbp vs 32 kbp tuned).

## Evidence

- [readme] Normally with ~50x coverage, NodeCovTh 1 EdgeCovTh 0 can produce nice results.: "Normally with ~50x coverage, NodeCovTh 1 EdgeCovTh 0 can produce nice results."
- [readme] The previous computations can be loaded and two parameters can be fine-tuned to construct a cleaner de Bruijn/ k-mer graph: "The previous computations can be loaded and two parameters can be fine-tuned to construct a cleaner de Bruijn/ k-mer graph"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs"
- [results] We used QUAST 3.032 in its default setting to evaluate the assembly results: "We used QUAST 3.032 in its default setting to evaluate the assembly results"
