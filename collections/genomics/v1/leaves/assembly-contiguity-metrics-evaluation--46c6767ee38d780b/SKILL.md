---
name: assembly-contiguity-metrics-evaluation
description: Use when a hybrid or long-read assembly (backbone_raw.fasta, polished
  consensus) has been produced and you must check N50 before proceeding to consensus
  tuning or final reporting.
when_to_use_negative:
- The contigs are raw SparseAssembler DBG contigs that skip gap-closing/scaffolding
  — these intermediate Contigs.txt outputs have their own (much lower) N50 (~29-32
  kbp) and are not the assembly to benchmark
- You need base-level accuracy/error rates — this skill covers contiguity (N50/NG50)
  only; base quality is assessed separately via BLASR+Sparc consensus polishing
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: AssemblyStatistics
  role: Computes essential assembly statistics (including N50) from a contigs FASTA;
    used to gate whether to proceed past the DBG2OLC backbone stage
  repo: https://github.com/yechengxi/AssemblyUtility
- name: QUAST 3.0
  role: Evaluates final assembly results in its default setting
- name: DBG2OLC
  role: Produces the backbone_raw.fasta whose N50 is the quantity being evaluated;
    parameter tuning (KmerCovTh, MinOverlap, AdaptiveTh) is driven by the measured
    N50
  repo: https://github.com/yechengxi/DBG2OLC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:5a49ead12abc0f5dab247809f53988778c3a5ce4d7f0aced94f8b571d663416f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/AssemblyUtility
  tools:
  - AssemblyStatistics
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# assembly-contiguity-metrics-evaluation

## Summary

Evaluate de novo genome assembly contiguity by computing N50/NG50 statistics with AssemblyStatistics or QUAST 3.0, using the DBG2OLC paper's S.cer w303 yeast benchmark (backbone N50 = 583 kbp from 20x PacBio + Illumina contigs) as the reference outcome.

## When to use

Use when a hybrid or long-read assembly (backbone_raw.fasta, polished consensus) has been produced and you must check N50 before proceeding to consensus tuning or final reporting.

## When NOT to use

- The contigs are raw SparseAssembler DBG contigs that skip gap-closing/scaffolding — these intermediate Contigs.txt outputs have their own (much lower) N50 (~29-32 kbp) and are not the assembly to benchmark
- You need base-level accuracy/error rates — this skill covers contiguity (N50/NG50) only; base quality is assessed separately via BLASR+Sparc consensus polishing

## Inputs

- Draft assembly FASTA (e.g. backbone_raw.fasta from DBG2OLC or polished consensus output)
- Optional reference genome FASTA for QUAST-based NG50 evaluation (e.g. NCBI GCA_000292815.1 454 yeast assembly)
- Estimated genome size (GS 12000000 for S.cer w303)

## Outputs

- Assembly statistics txt files (two per AssemblyStatistics run) containing contiguity metrics including N50
- QUAST 3.0 evaluation report of final assembly N50
- Metrics-file entry recording the achieved N50 (e.g. 583 kbp backbone)

## How to apply

After each DBG2OLC overlap/layout run, evaluate the backbone (backbone_raw.fasta) with ./AssemblyStatistics contigs YourAssembly.fasta, which writes two txt files of essential assembly statistics; the README explicitly instructs to check the backbone N50 and 'keep tuning and don't proceed' (e.g. retune KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x-20x PacBio) if unsatisfied. For final polished assemblies, evaluate with QUAST 3.0 in its default setting. On the w303 yeast dataset (GS 12000000), the expected backbone N50 after Step2 is 583 kbp using k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1. Use dot-plot-based structural checks (AssemblyUtility) as a complementary sanity check that long reads align diagonally to the assembly.

## Related tools

- **AssemblyStatistics** (Computes essential assembly statistics (including N50) from a contigs FASTA; used to gate whether to proceed past the DBG2OLC backbone stage) — https://github.com/yechengxi/AssemblyUtility
- **QUAST 3.0** (Evaluates final assembly results in its default setting)
- **DBG2OLC** (Produces the backbone_raw.fasta whose N50 is the quantity being evaluated; parameter tuning (KmerCovTh, MinOverlap, AdaptiveTh) is driven by the measured N50) — https://github.com/yechengxi/DBG2OLC

## Examples

```
./AssemblyStatistics contigs YourAssembly.fasta && quast.py polished_consensus.fasta
```

## Evaluation signals

- w303 yeast backbone with k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 should give N50 ≈ 583 kbp
- Reported hybrid result: 10x-20x PacBio N50s are >10x better than Illumina-only assemblies (Illumina contig N50 ~29-32 kbp)
- AssemblyStatistics run must generate two txt files of statistics for the supplied FASTA
- Structural correctness check: nearly perfect diagonal dot plots of long reads vs assembly indicate a structurally correct assembly from as low as 10x long reads
- If backbone N50 is unsatisfactory, the workflow requires retuning DBG2OLC parameters before the consensus step — proceeding without this gate is a process violation

## Limitations

- Comparisons of other programs' performance on genomes >100 Mbp are not reported in the paper's results section; readers are directed to the original publications
- N50 alone does not guarantee base-level accuracy — the paper separates structural correctness (dot plots, N50) from consensus polishing quality
- README warns that combining DBG2OLC with other assembly/error-correction software may produce worse results because such tools introduce errors, so benchmark values are only comparable within the prescribed pipeline

## Evidence

- [readme] In our test run, the N50 is 583kbp.: "In our test run, the N50 is 583kbp."
- [readme] You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed.: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
- [results] We used QUAST 3.032 in its default setting to evaluate the assembly results: "We used QUAST 3.032 in its default setting to evaluate the assembly results"
- [results] Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone: "Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone"
- [readme] The program will generate two txt files containing essential statistics about your assembly.: "The program will generate two txt files containing essential statistics about your assembly."
