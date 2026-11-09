---
name: minion-trained-alignment-parameter-application
description: Use when aligning MinION nanopore reads (FASTA/FASTQ) against a target
  genome and you need MinION-trained LAST scoring parameters plus split-alignment
  handling for chimeric or rearranged reads.
when_to_use_negative:
- The input is not Oxford Nanopore/MinION data — the parameters are last-train-derived
  for MinION technology specifics and are not transferable to Illumina or PacBio reads
- The target sequence database has not been created/indexed with lastdb — NanoPipe
  requires a prepared target before alignment
- Metagenomic analysis is the goal — the article states metagenomics analysis is not
  yet supported and implementation is planned
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: LAST 946
  role: Sequence aligner used to map MinION reads to the target using MinION-trained
    parameters and substitution matrices
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-split
  role: Resolves split best-scoring alignments into most probable part-wise alignments
    for rearrangements, gene fusions, and chimeric sequences
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-train
  role: Derives the MinION-specific alignment parameters and substitution matrices
    applied by lastal
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: NanoPipe
  role: Web-driven automatic pipeline that orchestrates the alignment stage and reports
    the LAST parameters and mapping statistics
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:9f06cd5da23047e69c602249250ac9499bd2902dadb6be7de16c1ef040c773f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  - last-split
  - last-train
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# minion-trained-alignment-parameter-application

## Summary

Maps MinION sequencing reads to a target genome using the LAST aligner (version 946) with last-train-derived, MinION-adjusted scoring parameters and a substitution matrix, then resolves split alignments with last-split. This is the mapping stage of the NanoPipe pipeline, tuned to the error profile of Oxford Nanopore data.

## When to use

Use when aligning MinION nanopore reads (FASTA/FASTQ) against a target genome and you need MinION-trained LAST scoring parameters plus split-alignment handling for chimeric or rearranged reads.

## When NOT to use

- The input is not Oxford Nanopore/MinION data — the parameters are last-train-derived for MinION technology specifics and are not transferable to Illumina or PacBio reads
- The target sequence database has not been created/indexed with lastdb — NanoPipe requires a prepared target before alignment
- Metagenomic analysis is the goal — the article states metagenomics analysis is not yet supported and implementation is planned

## Inputs

- MinION query reads in FASTA or FASTQ format (e.g., 21 MB Plasmodium query, or 900 MB raw 1D reads)
- Pre-built lastdb index of the target genome (e.g., Plasmodium falciparum 3D7, hg38, or custom target sequence)
- last-train-derived alignment parameters and substitution matrix (input.substmatrix)

## Outputs

- calc.lastalign.maf — alignments of query reads to the target in MAF format
- calc.statistics — mapping statistics reported by NanoPipe (e.g., total mapped reads)
- calc.targetcounts — read distribution per chromosome/scaffold
- calc.querylens — query read length statistics

## How to apply

First build or reuse a pre-built lastdb index of the target genome (e.g., NANOPIPE/targets/plasmodium/target). Run lastal (LAST version 946) with the MinION-trained parameters produced by a prior last-train step, using the NanoPipe invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix' against the target index and the query reads. Pipe the lastal output through last-split so reads whose best-scoring alignment is split across different target regions (rearrangements, gene fusions, chimeric sequences) receive their most probable part-wise alignments. Save the alignments in MAF format as calc.lastalign.maf, and record alignment statistics into calc.statistics, calc.targetcounts, and calc.querylens. Optionally set a read length limit (e.g., 200 nucleotides) to avoid mapping of too-short 'noise' reads.

## Related tools

- **LAST 946** (Sequence aligner used to map MinION reads to the target using MinION-trained parameters and substitution matrices) — https://github.com/IOB-Muenster/nanopipe2
- **last-split** (Resolves split best-scoring alignments into most probable part-wise alignments for rearrangements, gene fusions, and chimeric sequences) — https://github.com/IOB-Muenster/nanopipe2
- **last-train** (Derives the MinION-specific alignment parameters and substitution matrices applied by lastal) — https://github.com/IOB-Muenster/nanopipe2
- **NanoPipe** (Web-driven automatic pipeline that orchestrates the alignment stage and reports the LAST parameters and mapping statistics) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
lastal -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix NANOPIPE/targets/plasmodium/target query.fastq | last-split > calc.lastalign.maf
```

## Evaluation signals

- The 'Align with last' step of the NANOPIPE/check.sh test case completes successfully (documented at ~59 seconds) and produces a non-empty calc.lastalign.maf
- calc.statistics, calc.targetcounts, and calc.querylens files are generated and populated
- Mapping rate matches reported behavior — e.g., 99.9% of query reads mapped in the Plasmodium study case
- Reported parameters match the NanoPipe invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix'
- Alignments are valid MAF and per-chromosome/scaffold counts in calc.targetcounts sum to the total mapped reads

## Limitations

- The substitution matrix and scoring parameters are precompiled from last-train for MinION data; applying them to non-ONT data is not supported by the pipeline
- Reads whose best-scoring alignment is split are assigned part-wise alignments, which may misrepresent multi-mapping reads if the target has extensive repeats
- The documented test-case runtime (~59 seconds for the Plasmodium alignment step) scales with target size — the 900 MB human 1D-read analysis took 135 minutes

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [abstract] NanoPipe developed with MinION-adjusted alignment parameters: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
- [methods] Workflow step: alignment of MinION reads against the target sequences: "(2) alignment of MinION reads against the target sequences"
- [results] Plasmodium study case mapping rate: "NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations"
- [results] Read length limit to avoid noise: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
