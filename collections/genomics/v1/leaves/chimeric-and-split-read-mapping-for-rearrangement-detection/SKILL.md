---
name: chimeric-and-split-read-mapping-for-rearrangement-detection
description: Use when aligning Oxford Nanopore MinION reads against a target sequence
  and reads may map to multiple or disjoint target regions (rearrangements, fusions,
  chimeras), or when reporting per-chromosome mapping statistics is required.
when_to_use_negative:
- The downstream goal is metagenomics analysis — NanoPipe does not yet support it
  (implementation is planned future work)
- The reads are not nanopore MinION reads — alignment parameters are specifically
  adjusted for MinION sequencing technology specifics
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LAST 946
  role: Performs the mapping stage, aligning MinION query reads against the lastdb
    target index with last-train-derived MinION parameters
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-split
  role: Post-processes lastal output to assign reads' most probable part-wise alignments
    when their best-scoring alignment is split across target regions
- name: last-train
  role: Supplies the precompiled MinION-specific alignment parameters and substitution
    matrices used by lastal
- name: NanoPipe
  role: Web-driven pipeline that orchestrates the alignment stage and reports mapping
    statistics after the completed analysis
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:49c9a1900ca7b29cfcaa06763e16f970d56b30830ac7140e7a3bd77c1ee2abd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  - last-split
  - last-train
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# chimeric-and-split-read-mapping-for-rearrangement-detection

## Summary

Maps MinION nanopore query reads to a target genome with the LAST aligner (lastal) using last-train-derived parameters, then pipes the output through last-split so reads whose best-scoring alignment is split across target regions receive their most probable part-wise alignments, handling rearrangements, gene fusions, and chimeric sequences.

## When to use

Use when aligning Oxford Nanopore MinION reads against a target sequence and reads may map to multiple or disjoint target regions (rearrangements, fusions, chimeras), or when reporting per-chromosome mapping statistics is required.

## When NOT to use

- The downstream goal is metagenomics analysis — NanoPipe does not yet support it (implementation is planned future work)
- The reads are not nanopore MinION reads — alignment parameters are specifically adjusted for MinION sequencing technology specifics

## Inputs

- MinION query reads in FASTA or FASTQ format
- Pre-built lastdb index of the target genome (e.g., NANOPIPE/targets/plasmodium/target)
- last-train-derived alignment parameters and substitution matrix (input.substmatrix)

## Outputs

- calc.lastalign.maf (split-read alignments in MAF format)
- calc.statistics (mapping statistics: total mapped reads)
- calc.targetcounts (per-chromosome/scaffold read distribution)
- calc.querylens (query read length information)
- LAST parameters used for the alignment

## How to apply

Build or reuse a pre-built lastdb index of the target genome (e.g., NANOPIPE/targets/plasmodium/target). Run lastal (LAST version 946) with MinION-trained parameters from the preceding last-train step, using the NanoPipe invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix' against the target index and the query reads. Pipe the lastal output through last-split so that reads whose best-scoring alignment is split into parts mapped to different target regions are assigned their most probable part-wise alignments. Save the alignments in MAF format as calc.lastalign.maf and record the mapping statistics NanoPipe reports (calc.statistics, calc.targetcounts, calc.querylens), including total mapped reads and their distribution per chromosome/scaffold. Optionally apply a user-set read length limit (e.g., 200 nucleotides) to avoid mapping of too-short reads.

## Related tools

- **LAST 946** (Performs the mapping stage, aligning MinION query reads against the lastdb target index with last-train-derived MinION parameters) — https://github.com/IOB-Muenster/nanopipe2
- **last-split** (Post-processes lastal output to assign reads' most probable part-wise alignments when their best-scoring alignment is split across target regions)
- **last-train** (Supplies the precompiled MinION-specific alignment parameters and substitution matrices used by lastal)
- **NanoPipe** (Web-driven pipeline that orchestrates the alignment stage and reports mapping statistics after the completed analysis) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
lastal -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix NANOPIPE/targets/plasmodium/target reads.fa | last-split > calc.lastalign.maf
```

## Evaluation signals

- The alignment stage of the check.sh test case completes successfully (the documented test case finishes the 'Align with last' step in ~59 seconds)
- calc.lastalign.maf is produced and is non-empty
- The reported fraction of mapped query reads is consistent with the Plasmodium study case, in which NanoPipe mapped 99.9% of all query reads
- calc.statistics reports a total mapped-read count, and calc.targetcounts gives a per-chromosome/scaffold distribution
- The reported LAST parameters match the NanoPipe invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix'

## Limitations

- Mapping statistics describe the mechanism rather than a separately validated benchmark of split-alignment accuracy
- A read length limit (e.g., 200 nucleotides) may be needed to avoid 'noise' from mapping of too-short reads
- More sophisticated statistics for SNP detection and evaluation are planned future work; current SNP detection statistics are not yet sophisticated

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [other] last-split assigns most probable part-wise alignments: [evidence span withheld — anchor status: not_found]
- [results] user-set read length limit to avoid short-read noise: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
- [readme] readme: NanoPipe provides alignments to any target of interest: "As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
