---
name: assembly-contiguity-statistics
description: Use when you have a draft assembly FASTA (e.g. DBG2OLC's backbone_raw.fasta)
  and need N50/NG50 contiguity figures to decide whether to tune parameters, proceed
  to consensus, or compare against published metrics.
when_to_use_negative:
- The contigs are not raw DBG contigs — the README warns that contigs processed by
  repeat-resolving techniques such as gap closing or scaffolding introduce errors
  and can yield poor downstream results, so statistics on such input do not reflect
  the intended pipeline stage
- The goal is base-level accuracy after polishing, not draft contiguity — the consensus
  step (~90% of DBG2OLC runtime, ~2000 CPU hours on human) is a separate evaluation
  problem
- You intend to change the assembly pipeline itself rather than measure its output;
  this skill only reports statistics and gives no tuning advice
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: AssemblyUtility (AssemblyStatistics)
  role: Generates two txt files of essential assembly statistics (N50/NG50) from a
    draft assembly FASTA; also provides SelectLongestReads for preparing the 30x PacBio
    subset whose assembly N50 is being evaluated
  repo: https://github.com/yechengxi/AssemblyUtility
- name: QUAST 3.0
  role: Independent evaluation of assembly results in its default setting, used in
    the paper to assess contiguity
- name: DBG2OLC
  role: Produces the draft backbone_raw.fasta whose N50/NG50 are the metrics being
    reported (N50 = 6 Mbp on 30x human PacBio reads; NG50 = 433 kbp from 10x)
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles the 50x Illumina reads (k 51, g 15, NodeCovTh 1, EdgeCovTh 0)
    into the accurate short contigs used in the hybrid assembly whose contiguity is
    measured
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:2bd6b30dd33eee9de3828be958b0b0bb976454ed64e75e1fb77775cdad4c7cf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/AssemblyUtility
  tools:
  - AssemblyUtility
  - AssemblyUtility (AssemblyStatistics)
  - DBG2OLC
  - QUAST 3.0
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# assembly-contiguity-statistics

## Summary

Quantifies the contiguity of a draft genome assembly (N50, NG50 and related statistics) using AssemblyUtility's AssemblyStatistics, optionally cross-checked with QUAST 3.0, to judge whether an assembly stage (e.g. the unpolished DBG2OLC backbone) is good enough before investing in consensus/polishing.

## When to use

Use when you have a draft assembly FASTA (e.g. DBG2OLC's backbone_raw.fasta) and need N50/NG50 contiguity figures to decide whether to tune parameters, proceed to consensus, or compare against published metrics.

## When NOT to use

- The contigs are not raw DBG contigs — the README warns that contigs processed by repeat-resolving techniques such as gap closing or scaffolding introduce errors and can yield poor downstream results, so statistics on such input do not reflect the intended pipeline stage
- The goal is base-level accuracy after polishing, not draft contiguity — the consensus step (~90% of DBG2OLC runtime, ~2000 CPU hours on human) is a separate evaluation problem
- You intend to change the assembly pipeline itself rather than measure its output; this skill only reports statistics and gives no tuning advice

## Inputs

- Draft assembly FASTA (e.g. backbone_raw.fasta from DBG2OLC, or Contigs.txt from SparseAssembler)
- Optional estimated genome size in bp (e.g. ~3 Gbp for H. sapiens) for NG50 calculation

## Outputs

- Two txt files from AssemblyStatistics containing essential assembly statistics (contig count, N50, NG50, total length)
- QUAST 3.0 report when used as the cross-check evaluator

## How to apply

Run AssemblyUtility's AssemblyStatistics on the draft FASTA (e.g. backbone_raw.fasta produced by DBG2OLC's overlap/layout stage), optionally passing an estimated genome size to obtain NG50 in addition to N50; the program generates two txt files with the essential assembly statistics. Evaluate the draft (unpolished) assembly before the BLASR/Sparc consensus step — the DBG2OLC manual explicitly says to check the N50 of backbone_raw.fasta and keep tuning parameters (k 17, KmerCovTh, MinOverlap, AdaptiveTh, RemoveChimera) if unsatisfied rather than proceed. Benchmark against the paper's human figures: N50 = 6 Mbp for the draft from the longest 30x PacBio reads plus 50x Illumina SparseAssembler contigs (k 51, g 15, NodeCovTh 1, EdgeCovTh 0), and NG50 = 433 kbp starting from 10x PacBio data. QUAST 3.0 in its default setting can be used as an independent evaluator of the same assembly.

## Related tools

- **AssemblyUtility (AssemblyStatistics)** (Generates two txt files of essential assembly statistics (N50/NG50) from a draft assembly FASTA; also provides SelectLongestReads for preparing the 30x PacBio subset whose assembly N50 is being evaluated) — https://github.com/yechengxi/AssemblyUtility
- **QUAST 3.0** (Independent evaluation of assembly results in its default setting, used in the paper to assess contiguity)
- **DBG2OLC** (Produces the draft backbone_raw.fasta whose N50/NG50 are the metrics being reported (N50 = 6 Mbp on 30x human PacBio reads; NG50 = 433 kbp from 10x)) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles the 50x Illumina reads (k 51, g 15, NodeCovTh 1, EdgeCovTh 0) into the accurate short contigs used in the hybrid assembly whose contiguity is measured) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./AssemblyStatistics contigs backbone_raw.fasta
```

## Evaluation signals

- AssemblyStatistics produces exactly two txt output files containing the essential statistics for the input FASTA
- For the H. sapiens 30x run (longest 30x PacBio reads + 50x Illumina contigs), the unpolished backbone_raw.fasta N50 matches the paper's ~6 Mbp draft figure
- For the 54x human dataset starting from 10x PacBio data, the NG50 reaches 433 kbp
- Dot-plot sanity check: nearly perfect diagonal plots against the reference indicate the measured contiguity reflects structurally correct assembly (as low as 10x long read data)
- Cross-checking the same FASTA with QUAST 3.0 in default setting yields consistent contiguity metrics

## Limitations

- Most competing assemblers do not scale linearly with data size (thousands of CPU hours per run on genomes >100 Mbp), so direct metric comparisons at scale are limited and readers are directed to original publications
- The human consensus/polishing step takes roughly 2000 CPU hours versus 405,000 CPU hours for the overlapping process in the initial PacBio report — draft N50 alone does not capture the cost of reaching a polished assembly
- The 17-mer index for the 30x human overlap/layout stage requires ~70 GB peak memory and ~37 CPU hours, so reproducing the reported metrics needs a machine with sufficient RAM

## Evidence

- [readme] The program will generate two txt files containing essential statistics about your assembly.: "The program will generate two txt files containing essential statistics about your assembly."
- [results] DBG2OLC is able to produce an assembly with high contiguity starting from 10x PacBio data (NG50 433 kbp): "DBG2OLC is able to produce an assembly with high contiguity starting from 10x PacBio data (NG50 433 kbp)"
- [results] We used QUAST 3.0 in its default setting to evaluate the assembly results: "We used QUAST 3.032 in its default setting to evaluate the assembly results"
- [readme] You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed.: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
