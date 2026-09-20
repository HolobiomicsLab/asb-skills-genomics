---
name: perl-pipeline-parallel-execution
description: Use when a Perl genotyping pipeline must test thousands of indel sites
  independently and per-site forked parallelism (Parallel::ForkManager, THREADS from
  parameters.ini) is needed to keep runtime tractable.
when_to_use_negative:
- The population is small or the site count is low enough that a serial run completes
  quickly — forking overhead and 20 threads are unnecessary.
- The task needs shared mutable state across sites; the ForkManager model forks independent
  children per site, so results must be written to per-type files rather than accumulated
  in-memory.
- Perl and the required modules (Parallel::ForkManager, Getopt::Long, Statistics::Distributions)
  cannot be installed in the target environment — the pipeline depends on them via
  cpan.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: Perl pipeline whose third step (segregation analysis) is parallelized per
    indel site with Parallel::ForkManager using THREADS 20
  repo: https://github.com/tongchf/InDelGT
- name: Parallel::ForkManager
  role: Perl module used to fork per-site child processes across THREADS workers during
    segregation analysis
- name: Getopt::Long
  role: Perl module that parses InDelGT.pl command-line options (-o directory, --nocatalog/--nocall/--nofilter,
    -p population type, --help)
- name: Statistics::Distributions
  role: Perl module used inside each forked worker for the segregation-ratio test
    against expected Mendelian ratios
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:13a33f1cac4b6ba30760527e5363ecbb44c99c974f2601ac5b2c76d5eae77c68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/tongchf/InDelGT
  repo_url: https://github.com/tongchf/InDelGT
  tools:
  - Getopt::Long
  - InDelGT
  - Parallel::ForkManager
  - Statistics::Distributions
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# perl-pipeline-parallel-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parallelize the per-site InDelGT segregation analysis in Perl using Parallel::ForkManager with THREADS 20, so Mendelian segregation tests across 50,532 cataloged indel sites in the F1 Populus population finish in hours rather than days.

## When to use

Use when a Perl genotyping pipeline must test thousands of indel sites independently and per-site forked parallelism (Parallel::ForkManager, THREADS from parameters.ini) is needed to keep runtime tractable.

## When NOT to use

- The population is small or the site count is low enough that a serial run completes quickly — forking overhead and 20 threads are unnecessary.
- The task needs shared mutable state across sites; the ForkManager model forks independent children per site, so results must be written to per-type files rather than accumulated in-memory.
- Perl and the required modules (Parallel::ForkManager, Getopt::Long, Statistics::Distributions) cannot be installed in the target environment — the pipeline depends on them via cpan.

## Inputs

- parameters.ini parameter file ([folders], [parameters], [data files] sections) including THREADS setting
- Reference genome FASTA (reference.fasta)
- Paired-end FASTQ files of 2 parents and 20 progeny (e.g. female.R1.fq/female.R2.fq, sample01.R1.fq/sample01.R2.fq)
- InDel genotype data for parents and progeny produced by the InDelGT genotyping step (for the segregation-analysis stage)

## Outputs

- Per-segregation-type result files for CP populations: aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt
- Mendelian-filtered marker sets (e.g. 1539 ab × aa, 1262 aa × ab, 3 ab × ab, 1 ab × ac; 2805 of 50,532 sites passing P > 0.01 in the example)

## How to apply

Configure the 'parameters' section of parameters.ini, setting THREADS:20 alongside the analysis thresholds (GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, PROGENY_NUMBER:20). Load the Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions (installable via cpan), then run the whole three-step calculation (catalog, genotyping, segregation analysis) with a single command; Getopt::Long parses options such as -o <directory> and the population type (-p CP;BC1;BC2;F2, default CP). Use Parallel::ForkManager to fork one child per THREADS worker so each indel site's segregation-ratio test (Statistics::Distributions, rejecting PVALUE < 0.01) and per-type result file writing (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt for CP) proceeds concurrently. Because a practical run takes hours or days, launch the command in the background with nohup ... &. Skip completed stages on re-run with --nocatalog, --nocall, or --nofilter rather than recomputing everything.

## Related tools

- **InDelGT** (Perl pipeline whose third step (segregation analysis) is parallelized per indel site with Parallel::ForkManager using THREADS 20) — https://github.com/tongchf/InDelGT
- **Parallel::ForkManager** (Perl module used to fork per-site child processes across THREADS workers during segregation analysis)
- **Getopt::Long** (Perl module that parses InDelGT.pl command-line options (-o directory, --nocatalog/--nocall/--nofilter, -p population type, --help))
- **Statistics::Distributions** (Perl module used inside each forked worker for the segregation-ratio test against expected Mendelian ratios)

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -p CP -o directory &
```

## Evaluation signals

- Thread count in parameters.ini matches THREADS:20 and PROGENY_NUMBER:20 with the actual number of progeny FASTQ pairs listed.
- For the CP example data, per-type output file line counts reproduce 1539 ab × aa, 1262 aa × ab, 3 ab × ab, 1 ab × ac (2805 total of 50,532 cataloged indels passing P > 0.01).
- Every retained site has a segregation-test P-value ≥ 0.01 as enforced by PVALUE:0.01; sites below threshold are absent from the seven output files.
- The nohup background run completes without forking/module errors (all three Perl modules load), and re-runs with --nocatalog/--nocall/--nofilter skip the already-finished steps.

## Limitations

- A practical run takes 'quite a few hours or even several days', so the command must be run in the background (nohup ... &) and interrupted serial runs are impractical.
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth, which can degrade per-site genotypes feeding the parallel step.
- Determining heterozygotes requires a higher coverage depth than homozygotes, so low-depth progeny sites may fail the HETEROZYGOUS_DEPTH:3 threshold before the segregation test.
- No version number is stated for InDelGT in the article text, so the parallel implementation details are only documented via the GitHub READMEs and parameters.ini.

## Evidence

- [results] Parallelize the per-site computation with Parallel::ForkManager using THREADS 20.: [evidence span withheld — anchor status: not_found]
- [results] Test each site's segregation ratio against the expected Mendelian ratios for the seven cross-pollination (CP) segregation types with the Statistics::Distributions module: [evidence span withheld — anchor status: not_found]
