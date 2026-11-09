---
name: long-read-overlap-layout-assembly
description: Use when assembling a genome from low-coverage (10x–20x) long erroneous
  reads (PacBio/3GS) together with preassembled high-coverage (e.g., 50x) Illumina
  DBG contigs, and N50 far beyond short-read-only assembly is the goal.
when_to_use_negative:
- You have no accurate short-read contigs and intend a purely long-read assembly —
  the DBG2OLC manual warns combining DBG2OLC with other error-correction tools 'may
  produce worse results'
- Input contigs come from a short-read assembler run with repeat-resolving steps such
  as gap closing or scaffolding, which introduce errors that degrade the final assembly
- Backbone N50 after Step 2 is unsatisfactory — the README says to keep tuning (KmerCovTh/MinOverlap/AdaptiveTh)
  and 'don't proceed' to consensus
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: 'Core overlap-layout tool: compresses long reads via shared k-mers with the
    DBG contigs and lays out contigs into backbone_raw.fasta'
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles 50x Illumina short reads into accurate raw DBG contigs (Contigs.txt)
    used as anchors
  repo: https://github.com/yechengxi/SparseAssembler
- name: BLASR
  role: Aligns all raw PacBio reads to the assembly backbone during the consensus
    step (--minMatch 19, -m 5)
- name: Sparc
  role: Sparsity-based consensus module that polishes the backbone using BLASR m5
    mappings, invoked via split_and_run_sparc.sh
  repo: https://github.com/yechengxi/Sparc
- name: AssemblyUtility
  role: Provides SelectLongestReads for read subsetting and AssemblyStatistics for
    N50/assembly evaluation
  repo: https://github.com/yechengxi/AssemblyUtility
- name: QUAST 3.0
  role: Evaluates final assembly quality (N50) in its default setting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:7dcef5eb2c48df0e8e2b008e39324bf18cfc844e6f1ea07c450ac7444d478dc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# long-read-overlap-layout-assembly

## Summary

Hybrid overlap-layout assembly using DBG2OLC: compress erroneous long reads (PacBio) onto accurate short-read DBG contigs to build a backbone, then polish with BLASR + Sparc consensus. Achieves high contiguity (e.g., yeast N50 583 kbp) from only 10x–20x long-read coverage.

## When to use

Use when assembling a genome from low-coverage (10x–20x) long erroneous reads (PacBio/3GS) together with preassembled high-coverage (e.g., 50x) Illumina DBG contigs, and N50 far beyond short-read-only assembly is the goal.

## When NOT to use

- You have no accurate short-read contigs and intend a purely long-read assembly — the DBG2OLC manual warns combining DBG2OLC with other error-correction tools 'may produce worse results'
- Input contigs come from a short-read assembler run with repeat-resolving steps such as gap closing or scaffolding, which introduce errors that degrade the final assembly
- Backbone N50 after Step 2 is unsatisfactory — the README says to keep tuning (KmerCovTh/MinOverlap/AdaptiveTh) and 'don't proceed' to consensus

## Inputs

- High-coverage Illumina FASTQ reads (e.g., Illumina_50x.fastq, ~50x coverage)
- Low-coverage long PacBio reads in FASTA (e.g., Pacbio_20x.fasta, 10x–20x)
- Genome size estimate in bp (e.g., GS 12000000 for 12 Mbp yeast)

## Outputs

- SparseAssembler DBG contigs (Contigs.txt)
- DBG2OLC backbone (backbone_raw.fasta) and DBG2OLC_Consensus_info.txt
- Polished consensus assembly after Sparc (via split_and_run_sparc.sh)
- Assembly statistics (N50) from AssemblyStatistics or QUAST 3.0

## How to apply

First preassemble ~50x Illumina reads with SparseAssembler (e.g., k 51, g 15, NodeCovTh 1, EdgeCovTh 0, GS 12000000 for the S.cer w303 dataset), using raw DBG contigs (Contigs.txt) without gap-closing or scaffolding, since those heuristics introduce errors. Then run DBG2OLC in overlap-layout mode (e.g., k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f Pacbio_20x.fasta) to produce backbone_raw.fasta and DBG2OLC_Consensus_info.txt; check the backbone N50 and retune within suggested ranges (10x–20x PacBio: KmerCovTh 2–5, MinOverlap 10–30, AdaptiveTh 0.001–0.01) before proceeding. Finally, concatenate Contigs.txt with the PacBio reads into ctg_pb.fasta, raise the file-descriptor limit (ulimit -n unlimited), and run split_and_run_sparc.sh, which calls BLASR (--minMatch 19, -m 5) plus Sparc consensus to polish the backbone. Evaluate the final assembly with AssemblyStatistics or QUAST 3.0 and record N50.

## Related tools

- **DBG2OLC** (Core overlap-layout tool: compresses long reads via shared k-mers with the DBG contigs and lays out contigs into backbone_raw.fasta) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles 50x Illumina short reads into accurate raw DBG contigs (Contigs.txt) used as anchors) — https://github.com/yechengxi/SparseAssembler
- **BLASR** (Aligns all raw PacBio reads to the assembly backbone during the consensus step (--minMatch 19, -m 5))
- **Sparc** (Sparsity-based consensus module that polishes the backbone using BLASR m5 mappings, invoked via split_and_run_sparc.sh) — https://github.com/yechengxi/Sparc
- **AssemblyUtility** (Provides SelectLongestReads for read subsetting and AssemblyStatistics for N50/assembly evaluation) — https://github.com/yechengxi/AssemblyUtility
- **QUAST 3.0** (Evaluates final assembly quality (N50) in its default setting)

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f Pacbio_20x.fasta
```

## Evaluation signals

- Backbone N50 after Step 2 should reach the expected scale (e.g., 583 kbp on 20x PacBio S.cer w303); if unsatisfactory, retune KmerCovTh 2–5, MinOverlap 10–30, AdaptiveTh 0.001–0.01 before consensus
- Final polished N50 (AssemblyStatistics or QUAST 3.0) should be >10x better than the Illumina-only assembly (SparseAssembler contigs were ~29–32 kbp)
- Dot plots of the assembly against the reference should show a nearly perfect diagonal, indicating structurally correct assembly even from 10x long-read data
- SparseAssembler contigs must be raw DBG contigs — verify no gap-closing or scaffolding was applied before feeding Contigs.txt to DBG2OLC
- Consensus step completes without file-descriptor errors after 'ulimit -n unlimited'; ~90% of runtime is expected in the BLASR-based consensus step

## Limitations

- Sequencing-error correction tools or alternative pipelines combined with DBG2OLC may produce worse results; follow the prescribed steps before inventing your own pipeline
- Parameters are coverage-dependent: for 50x–100x PacBio the suggested ranges differ (KmerCovTh 2–10, MinOverlap 50–150, AdaptiveTh 0.01–0.02), and at ~100x ChimeraTh/ContigTh should be set to 2
- Most computation time (~90%) is spent on the BLASR-based consensus step, which dominates runtime (e.g., ~2000 CPU hours for the human assembly)
- Backbone from the first DBG2OLC run may be unsatisfactory and requires iterative retuning before proceeding to consensus

## Evidence

- [results] Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone: "Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone"
- [readme] raw DBG contigs without repeat resolving: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
- [readme] DBG2OLC w303 20x command: "./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta"
- [readme] In our test run, the N50 is 583kbp.: "In our test run, the N50 is 583kbp."
- [readme] follow the steps before you invent your own pipeline: "It is important to follow the steps before you invent your own pipeline. As combining DBG2OLC with other software tools may produce worse results."
- [results] SparseAssembler preassembly of Illumina reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [results] BLASR consensus step: "most of the computation time (~90%) is spent on the consensus step, in which BLASR28 is called to align all raw reads"
