---
name: genome-assembly-contiguity-and-error-evaluation
description: Use when you have a finished draft assembly (e.g. DBG2OLC_Consensus.fasta)
  plus a reference genome and need contiguity metrics (N50, NG50, contig count) or
  an error-rate estimate against the reference.
when_to_use_negative:
- The pipeline's overlap-layout backbone has an unsatisfactory N50 and parameters
  still need tuning — the README says to check the N50 'otherwise keep tuning and
  don't proceed' rather than evaluate prematurely
- The input is raw reads rather than an assembled FASTA — AssemblyStatistics evaluates
  assemblies, not sequencing data
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: AssemblyStatistics
  role: Computes contiguity statistics (two txt output files) of an assembly against
    a contigs reference, optionally with estimated genome size for NG50
  repo: https://github.com/yechengxi/AssemblyUtility
- name: QUAST 3.0
  role: Alternative evaluator run in its default setting to assess assembly results
    against the reference
- name: DBG2OLC
  role: Produces the backbone_raw.fasta and polished DBG2OLC_Consensus.fasta whose
    contiguity and error rate are being evaluated
  repo: https://github.com/yechengxi/DBG2OLC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:43753470adac386a3a6e69916ca40d6c8ae88df8afdd018becd96c72a4a33a02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/AssemblyUtility
  tools:
  - AssemblyStatistics
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# genome-assembly-contiguity-and-error-evaluation

## Summary

Evaluate de novo genome assemblies for contiguity and base-level accuracy using AssemblyStatistics and QUAST, e.g. confirming that a DBG2OLC hybrid assembly of the E. coli Oxford Nanopore MinION dataset resolves to a single contig with a polished error rate near 0.23%.

## When to use

Use when you have a finished draft assembly (e.g. DBG2OLC_Consensus.fasta) plus a reference genome and need contiguity metrics (N50, NG50, contig count) or an error-rate estimate against the reference.

## When NOT to use

- The pipeline's overlap-layout backbone has an unsatisfactory N50 and parameters still need tuning — the README says to check the N50 'otherwise keep tuning and don't proceed' rather than evaluate prematurely
- The input is raw reads rather than an assembled FASTA — AssemblyStatistics evaluates assemblies, not sequencing data

## Inputs

- Draft assembly FASTA (e.g. DBG2OLC_Consensus.fasta or backbone_raw.fasta)
- Reference genome FASTA (e.g. E. coli reference)
- Estimated genome size in bp (for NG50 computation)
- Unpolished backbone_raw.fasta plus DBG2OLC_Consensus_info.txt (for pre-consensus N50 checkpoint)

## Outputs

- Two statistics txt files from AssemblyStatistics (contig count, N50, NG50)
- QUAST evaluation report (default settings)
- Verification verdict: single-contig status and polished error rate (~0.23% for E. coli MinION)

## How to apply

Run AssemblyStatistics from the yechengxi/AssemblyUtility repo against the reference (optionally supplying the estimated genome size for NG50, e.g. './AssemblyStatistics contigs [YourAssembly.fasta] GS [Estimated_Genome_Size]'), which generates two txt files of assembly statistics. Alternatively, use QUAST 3.0 in its default setting, as the DBG2OLC authors did, to evaluate assembly results. For the E. coli MinION test case, check first that the polished assembly resolves to a single contig, then record the polished error rate, which should fall near the reported ~0.23%. After the overlap-layout step, evaluate the backbone N50 of backbone_raw.fasta with AssemblyStatistics before investing CPU time in the BLASR/Sparc consensus step — if the backbone N50 is unsatisfactory, go back and retune SparseAssembler/DBG2OLC parameters (k 17, AdaptiveTh, KmerCovTh, MinOverlap) rather than proceeding.

## Related tools

- **AssemblyStatistics** (Computes contiguity statistics (two txt output files) of an assembly against a contigs reference, optionally with estimated genome size for NG50) — https://github.com/yechengxi/AssemblyUtility
- **QUAST 3.0** (Alternative evaluator run in its default setting to assess assembly results against the reference)
- **DBG2OLC** (Produces the backbone_raw.fasta and polished DBG2OLC_Consensus.fasta whose contiguity and error rate are being evaluated) — https://github.com/yechengxi/DBG2OLC

## Examples

```
./AssemblyStatistics contigs DBG2OLC_Consensus.fasta
```

## Evaluation signals

- The E. coli MinION hybrid assembly evaluates as exactly one contig against the reference
- The polished DBG2OLC_Consensus.fasta error rate falls near the reported ~0.23%
- AssemblyStatistics produces its expected two txt statistics output files
- The backbone N50 of backbone_raw.fasta is checked and deemed satisfactory before running the ~90%-of-runtime BLASR consensus step
- QUAST runs cleanly with default settings and reports consistent contiguity metrics with AssemblyStatistics

## Limitations

- In DBG2OLC runs, ~90% of computation time is spent on the consensus step, so evaluating the backbone N50 early is critical to avoid wasted CPU hours on a poor layout
- The short-read contigs fed into DBG2OLC must be raw DBG contigs without gap closing or scaffolding, or the final results can be poor due to errors introduced by short-read assembly heuristics — evaluation of a scaffolded-input assembly may be misleading
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many such tools introduce errors

## Evidence

- [readme] The program will generate two txt files containing essential statistics about your assembly.: "The program will generate two txt files containing essential statistics about your assembly."
- [results] QUAST default evaluation: [evidence span withheld — anchor status: not_found]
- [results] Single-contig E. coli result: "DBG2OLC still successfully assembled the E. coli into one single contig"
- [readme] Backbone N50 checkpoint before consensus: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
- [readme] Raw contigs requirement: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
