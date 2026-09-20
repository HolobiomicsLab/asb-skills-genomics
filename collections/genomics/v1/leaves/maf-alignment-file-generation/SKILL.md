---
name: maf-alignment-file-generation
description: Use when MinION nanopore reads (FASTA/FASTQ) must be mapped to a target
  genome/reference and alignments need to be exported in MAF format with split-read
  handling for rearrangements or chimeric sequences.
when_to_use_negative:
- The task is metagenomics analysis of mixed-community reads — NanoPipe does not yet
  support metagenomics and its implementation was only planned per the discussion.
- The input is already an alignment file or the goal is consensus/polymorphism calling
  rather than read-to-target mapping — those are downstream NanoPipe stages with their
  own filter steps.
- A non-MAF alignment format (e.g., BAM/SAM) is strictly required without format conversion
  — this skill natively produces MAF output from lastal/last-split.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: LAST
  role: Sequence aligner (version 946, invoked as lastal) that maps MinION query reads
    against the lastdb target index using MinION-trained parameters and substitution
    matrices
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-split
  role: Post-processes lastal output to assign reads with split best-scoring alignments
    their most probable part-wise alignments before MAF output
- name: last-train
  role: Provides the alignment parameters and substitution matrices precompiled for
    MinION sequencing technology that lastal consumes
- name: NanoPipe
  role: Web-driven pipeline orchestrating the alignment stage, saving calc.lastalign.maf
    and the statistics files
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:b40f4eb8239f1f3e149df3ab5d73951861f160ea111ce190e41a76f540065f59
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# maf-alignment-file-generation

## Summary

Generates MAF-format alignments of MinION sequencing reads against a target genome using the LAST aligner (version 946) with MinION-trained parameters, piping lastal output through last-split to resolve split best-scoring alignments. This is the mapping stage of the NanoPipe pipeline that feeds downstream consensus and polymorphism analysis.

## When to use

Use when MinION nanopore reads (FASTA/FASTQ) must be mapped to a target genome/reference and alignments need to be exported in MAF format with split-read handling for rearrangements or chimeric sequences.

## When NOT to use

- The task is metagenomics analysis of mixed-community reads — NanoPipe does not yet support metagenomics and its implementation was only planned per the discussion.
- The input is already an alignment file or the goal is consensus/polymorphism calling rather than read-to-target mapping — those are downstream NanoPipe stages with their own filter steps.
- A non-MAF alignment format (e.g., BAM/SAM) is strictly required without format conversion — this skill natively produces MAF output from lastal/last-split.

## Inputs

- MinION query sequencing reads in FASTA or FASTQ format
- Pre-built lastdb index of the target genome/sequence (e.g., NANOPIPE/targets/plasmodium/target)
- last-train-derived substitution matrix (input.substmatrix) with MinION-adjusted alignment parameters

## Outputs

- calc.lastalign.maf — split-resolved read-to-target alignments in MAF format
- calc.statistics — alignment statistics reported to the user
- calc.targetcounts — distribution of mapped reads per chromosome/scaffold
- calc.querylens — query read length statistics

## How to apply

Build or reuse a lastdb index of the target genome (e.g., NANOPIPE/targets/plasmodium/target), then run lastal (LAST version 946) with MinION-trained parameters derived from last-train, using NanoPipe's invocation '-A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix' against the target index and query reads. Pipe the lastal output through last-split so reads whose best-scoring alignment is split across different target regions are assigned their most probable part-wise alignments, which handles rearrangements, gene fusions, and chimeric sequences. Save the resulting alignments in MAF format as calc.lastalign.maf, and record alignment statistics in calc.statistics, calc.targetcounts (mapped reads per chromosome/scaffold), and calc.querylens. Optionally set a read length limit (e.g., 200 nucleotides) to avoid mapping of too-short reads that add noise.

## Related tools

- **LAST** (Sequence aligner (version 946, invoked as lastal) that maps MinION query reads against the lastdb target index using MinION-trained parameters and substitution matrices) — https://github.com/IOB-Muenster/nanopipe2
- **last-split** (Post-processes lastal output to assign reads with split best-scoring alignments their most probable part-wise alignments before MAF output)
- **last-train** (Provides the alignment parameters and substitution matrices precompiled for MinION sequencing technology that lastal consumes)
- **NanoPipe** (Web-driven pipeline orchestrating the alignment stage, saving calc.lastalign.maf and the statistics files) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
lastal -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 -p input.substmatrix NANOPIPE/targets/plasmodium/target reads.fq | last-split > calc.lastalign.maf
```

## Evaluation signals

- The 'Align with last' step of the check.sh test case completes successfully (documented to finish in ~59 seconds).
- calc.lastalign.maf is produced and is non-empty.
- calc.statistics, calc.targetcounts, and calc.querylens are written alongside the MAF file.
- Mapped-read fraction is plausible for the study case — the Plasmodium example reported 99.9% of all query reads mapped.
- calc.targetcounts shows a sensible per-chromosome/scaffold distribution of mapped reads rather than all reads collapsing onto one contig.

## Limitations

- SNP detection statistics in NanoPipe are not yet sophisticated; more sophisticated statistics were planned as future work.
- Metagenomics analysis is not yet supported, limiting applicability to single-target alignment scenarios.
- Too-short reads can cause mapping 'noise'; a user-set read length limit (e.g., 200 nucleotides) is needed to mitigate this.
- The parameters used are precompiled MinION-trained values (LAST 946); they describe the mechanism rather than a separately validated benchmark, so retraining may be needed for other chemistries.

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [methods] alignment of MinION reads against the target sequences: "(2) alignment of MinION reads against the target sequences"
- [results] NanoPipe succeeded in mapping 99.9% of all query reads: "NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations"
- [results] read length limit can avoid noise from short reads: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
- [readme] NanoPipe provides alignments, statistics, and polymorphism information for MinION reads: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
