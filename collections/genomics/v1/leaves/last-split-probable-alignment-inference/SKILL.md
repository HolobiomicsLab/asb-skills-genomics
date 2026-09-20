---
name: last-split-probable-alignment-inference
description: Use when mapping Oxford Nanopore MinION reads to a target genome/sequence
  where reads may align to multiple loci or in split parts (rearrangements, fusions,
  chimeras), and a single -scoring alignment is insufficient.
when_to_use_negative:
- Input reads are not nanopore MinION data — NanoPipe's last-train-derived parameters
  are adjusted for MinION technology specifics
- The task is de novo assembly or metagenomics analysis, which NanoPipe does not support
  (metagenomics implementation is planned future work)
- Only the single best full-length alignment per read is needed and no split/multi-locus
  assignment is expected — plain lastal output without last-split suffices
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: LAST
  role: Sequence aligner (version 946) run via lastal with MinION-trained parameters
    to map reads against the target index
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-split
  role: Post-processes lastal output to assign reads their most probable part-wise
    alignments when the best alignment is split across target regions
- name: last-train
  role: Provides the precompiled alignment parameters and substitution matrices used
    by lastal for MinION data
- name: NanoPipe
  role: Web-driven pipeline orchestrating the mapping stage, saving MAF alignments
    and reporting mapping statistics to the user
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:7f6ad96f012565f0bbd805d71a41c317e615613052052060febb0e140b8207be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  - last-split
  - last-train
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# last-split-probable-alignment-inference

## Summary

Pipe LAST (version 946) lastal output through last-split so MinION reads whose best-scoring alignments are split across multiple target regions are assigned their most probable part-wise alignments, handling rearrangements, gene fusions, and chimeric sequences within NanoPipe's mapping stage.

## When to use

Use when mapping Oxford Nanopore MinION reads to a target genome/sequence where reads may align to multiple loci or in split parts (rearrangements, fusions, chimeras), and a single best-scoring alignment is insufficient.

## When NOT to use

- Input reads are not nanopore MinION data — NanoPipe's last-train-derived parameters are adjusted for MinION technology specifics
- The task is de novo assembly or metagenomics analysis, which NanoPipe does not support (metagenomics implementation is planned future work)
- Only the single best full-length alignment per read is needed and no split/multi-locus assignment is expected — plain lastal output without last-split suffices

## Inputs

- MinION query reads in FASTA or FASTQ format
- lastdb index of the target genome/sequence (e.g., NANOPIPE/targets/plasmodium/target)
- last-train-derived alignment parameters and substitution matrix (input.substmatrix)

## Outputs

- calc.lastalign.maf — alignments in MAF format
- calc.statistics — mapping statistics (total mapped reads)
- calc.targetcounts — read distribution per chromosome/scaffold
- calc.querylens — query read length statistics

## How to apply

First build (or reuse) a lastdb index of the target, e.g. NANOPIPE/targets/plasmodium/target. Run lastal with the MinION-trained parameters from the prior last-train step using the NanoPipe invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix' against the target index and the query reads. Pipe the lastal output through last-split, which reassigns each read to its most probable part-wise alignments rather than only its single best-scoring alignment. Save the result in MAF format as calc.lastalign.maf. Record the reported mapping statistics in calc.statistics, calc.targetcounts, and calc.querylens so the total mapped reads and per-chromosome/scaffold distribution can be reviewed by the user after the completed analysis.

## Related tools

- **LAST** (Sequence aligner (version 946) run via lastal with MinION-trained parameters to map reads against the target index) — https://github.com/IOB-Muenster/nanopipe2
- **last-split** (Post-processes lastal output to assign reads their most probable part-wise alignments when the best alignment is split across target regions)
- **last-train** (Provides the precompiled alignment parameters and substitution matrices used by lastal for MinION data)
- **NanoPipe** (Web-driven pipeline orchestrating the mapping stage, saving MAF alignments and reporting mapping statistics to the user) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
lastal -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix NANOPIPE/targets/plasmodium/target reads.fa | last-split > calc.lastalign.maf
```

## Evaluation signals

- The alignment stage of the NANOPIPE/check.sh test case ('Align with last') completes successfully (~59 seconds in the documented test case)
- calc.lastalign.maf exists and is non-empty
- Mapping statistics in calc.statistics show a high mapped-read fraction; the Plasmodium study case reported 99.9% of all query reads mapped
- calc.targetcounts shows a sensible per-chromosome/scaffold read distribution consistent with the expected target
- Reported LAST parameters match the trained MinION parameter set passed to lastal

## Limitations

- Alignment statistics in NanoPipe were described by mechanism rather than a separately validated benchmark
- Split/multi-mapping handling depends on last-split probabilistic assignment; reads shorter than a user-set read length limit (e.g., 200 nucleotides) may map as noise if not filtered
- Reported performance is dataset-dependent: ~4 minutes for a 21 MB FASTA query against P. falciparum versus 135 minutes for 900 MB of raw 1D reads against hg38

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [results] NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations: "NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations"
- [abstract] NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
- [results] To avoid 'noise,' i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
