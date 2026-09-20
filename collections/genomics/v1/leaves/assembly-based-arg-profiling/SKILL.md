---
name: assembly-based-arg-profiling
description: Reference for benchmarking read-based ARG profilers against assembly-based
  (metaFlye + Kraken2) and binning-based (SemiBin2 + GTDB-Tk) pipelines, or when coverage
  is high enough (≥8× for assembly, >32× for binning) to justify contig-based or MAG-based
  ARG host assignment.
when_to_use_negative:
- Coverage is low (below ~8× for assembly-based, below ~32× for binning-based profiling)
  — F1 stays at or below 0.5 and a read-based profiler such as Argo is preferred
- The goal is rapid ARG surveillance with species-level resolution without the compute
  and coverage cost of assembly and binning
- Input is already assembled contigs or MAGs — run the Kraken2/GTDB-Tk classification
  steps directly instead of re-assembling
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: metaFlye
  role: assembles downsampled Nanopore metagenomic reads into contigs using '--nano-raw
    --meta'
- name: Kraken2
  role: assigns GTDB taxonomy to assembled contigs
- name: SemiBin2
  role: bins contigs into MAGs with 'single_easy_bin --sequencing-type=long_read'
- name: GTDB-Tk
  role: labels MAGs with species-level taxonomy via 'classify_wf'
- name: DIAMOND
  role: maps reference genomes to SARG+ with blastx to compute expected ARG abundances
    for evaluation
- name: NanoSim
  role: simulates HQ and LQ Nanopore metagenomic reads for downsampled benchmarking
    coverages
- name: Argo
  role: read-based comparator whose detection thresholds (F1 0.811–0.935 at 1–2×)
    the assembly/binning baseline is measured against
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:ff4e01e2b4e0d3c9a1c9475371205189fa231005d6393c3ea0e3570830b069e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - GTDB-Tk
  - Kraken2
  - NanoSim
  - R
  - SemiBin2
  - metaFlye
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# assembly-based-arg-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarked strategy for ARG detection and host attribution that assembles long Nanopore reads into contigs (metaFlye), taxonomically classifies contigs (Kraken2), and optionally bins them into MAGs (SemiBin2 + GTDB-Tk), then compares the resulting ARG profiles against expected abundances. It serves as the assembly/binning-based baseline against which the read-based Argo profiler is measured across 1×–32× sequencing coverage.

## When to use

Reference for benchmarking read-based ARG profilers against assembly-based (metaFlye + Kraken2) and binning-based (SemiBin2 + GTDB-Tk) pipelines, or when coverage is high enough (≥8× for assembly, >32× for binning) to justify contig-based or MAG-based ARG host assignment.

## When NOT to use

- Coverage is low (below ~8× for assembly-based, below ~32× for binning-based profiling) — F1 stays at or below 0.5 and a read-based profiler such as Argo is preferred
- The goal is rapid ARG surveillance with species-level resolution without the compute and coverage cost of assembly and binning
- Input is already assembled contigs or MAGs — run the Kraken2/GTDB-Tk classification steps directly instead of re-assembling

## Inputs

- Quality-controlled Nanopore long reads (FASTQ/FASTA), e.g. NanoSim v3.1.0-simulated metagenomic samples downsampled to 1×, 2×, 4×, 8×, 16×, and 32× expected coverage
- GTDB reference taxonomy database for Kraken2 v2.1.3 and GTDB-Tk v2.4.0
- SARG+ ARG reference database (sarg.fa) for computing expected ARG abundances with DIAMOND
- Reference genomes of known sample members (e.g. 25 primary/opportunistic pathogens at even abundance)

## Outputs

- metaFlye assembly contigs (FASTA)
- Kraken2 taxonomic classifications of contigs
- SemiBin2 metagenomic-assembled genomes (MAGs) and GTDB-Tk species labels
- Type- and subtype-level ARG profiles per coverage point, evaluated with precision, recall, F1-score, Pearson correlation coefficient, and L1/L2 distances against expected abundances

## How to apply

Assemble downsampled Nanopore metagenomic reads with metaFlye v2.9.3 using '--nano-raw --meta' for metagenomic samples, then classify the resulting contigs with Kraken2 v2.1.3 against GTDB taxonomy. In parallel, bin contigs into MAGs with SemiBin2 v2.1.0 ('single_easy_bin --sequencing-type=long_read') and assign species-level labels to MAGs with GTDB-Tk v2.4.0 ('classify_wf'). Compute expected ARG abundances independently by mapping the reference genomes to SARG+ with DIAMOND blastx (--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hps 0). Evaluate both pipelines against expected abundances at ARG type and subtype levels using precision, recall, F1-score, Pearson correlation, and L1/L2 distances per coverage point (1×, 2×, 4×, 8×, 16×, 32×). The rationale is to quantify the coverage threshold at which assembly or binning recovers ARG-host links: assembly requires at least 8× and binning more than 32× coverage to exceed F1 of 0.9.

## Related tools

- **metaFlye** (assembles downsampled Nanopore metagenomic reads into contigs using '--nano-raw --meta')
- **Kraken2** (assigns GTDB taxonomy to assembled contigs)
- **SemiBin2** (bins contigs into MAGs with 'single_easy_bin --sequencing-type=long_read')
- **GTDB-Tk** (labels MAGs with species-level taxonomy via 'classify_wf')
- **DIAMOND** (maps reference genomes to SARG+ with blastx to compute expected ARG abundances for evaluation)
- **NanoSim** (simulates HQ and LQ Nanopore metagenomic reads for downsampled benchmarking coverages)
- **Argo** (read-based comparator whose detection thresholds (F1 0.811–0.935 at 1–2×) the assembly/binning baseline is measured against) — https://github.com/xinehc/argo

## Examples

```
metaFlye --nano-raw --meta downsampled_8x.fq -o asm && kraken2 --db gtdb --output kraken.tsv asm/assembly.fasta && SemiBin2 single_easy_bin --sequencing-type=long_read -i asm/assembly.fasta -o mags && GTDB-Tk classify_wf --mash_db gtdb mags -o gtdb_out
```

## Evaluation signals

- Assembly-based profiling reaches type-level F1 above 0.9 only at ≥8× coverage; binning-based profiling requires more than 32× coverage
- At 1–2× coverage, assembly- and binning-based methods remain at or below 0.5 F1 while Argo achieves F1 0.811–0.935 and precision 0.977–0.992
- Metrics table reproduces the monotonic improvement pattern in precision, recall, F1, Pearson correlation, and decreasing L1/L2 distances as coverage rises from 1× to 32×
- Expected ARG abundances are validated by DIAMOND blastx mapping of the 25 reference genomes to SARG+ (--evalue 1e-15 --id 90 --subject-cover 90)
- Kraken2 and GTDB-Tk outputs agree on GTDB species labels for the sample members present

## Limitations

- Assembly-based profiling needs at least 8× coverage and binning-based profiling more than 32× coverage to achieve F1-scores above 0.9
- Assembly- and binning-based methods are slower and coverage-hungrier than read-based profiling, though they may be better suited for de novo identification of ARG genomic contexts and hosts in new, understudied environments
- Benchmark depends on simulated reads (NanoSim HQ q=19.04 / LQ q=13.19) and even taxonomic abundance of 25 pathogens, which may not reflect real complex metagenomes

## Evidence
