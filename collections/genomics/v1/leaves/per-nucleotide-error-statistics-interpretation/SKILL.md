---
name: per-nucleotide-error-statistics-interpretation
description: Use when direct RNA nanopore reads aligned to a reference transcript
  (sorted/indexed BAM) must be scanned for position-specific error enrichment — e.g.
  elevated mismatch/deletion rates marking N1-methylpseudouridine sites in mRNA vaccines.
when_to_use_negative:
- Data are nanopore cDNA sequencing rather than direct RNA — the authors found cDNA
  sequencing more reliable, so error-based modification signatures at specific nucleosides
  are not the expected signal
- Input is raw FASTQ without a BAM alignment to a reference transcript — Mana analyses
  ONT alignments (.BAM), not unassembled reads
- Goal is general run/sample quality (read length N50, quality scores) rather than
  position-specific error signature — NanoPlot-style QC is the wrong lens here
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Mana
  role: Command-line tool run inside Docker (python3 mana.py --mrna) that analyses
    ONT alignments (.BAM) and emits the per-nucleotide error profile and mRNA length
    QC report
  repo: github.com/scchess/Mana
- name: Pysam
  role: Dependency used internally by Mana for BAM-level statistics on the aligned
    dRNA reads
- name: pysamstats
  role: Underlying utility (via Mana's dependency chain) for calculating per-genome-position
    variation statistics (matches, mismatches, deletions, insertions) from the SAM/BAM
    alignment
  repo: https://github.com/alimanfoo/pysamstats
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:7d27b2bd5101eb464f8360aa21a3e877d260f02f18018d9837fe583990acab10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mana
  - Pysam
  - pysamstats
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# per-nucleotide-error-statistics-interpretation

## Summary

Interprets the per-nucleotide error profile (mismatches, deletions, insertions) from direct RNA ONT alignments (.BAM) to detect chemically modified nucleosides such as N1-methylpseudouridine in synthetic mRNA, using Mana's QC reports (built on Pysam/pysamstats).

## When to use

Use when direct RNA nanopore reads aligned to a reference transcript (sorted/indexed BAM) must be scanned for position-specific error enrichment — e.g. elevated mismatch/deletion rates marking N1-methylpseudouridine sites in mRNA vaccines.

## When NOT to use

- Data are nanopore cDNA sequencing rather than direct RNA — the authors found cDNA sequencing more reliable, so error-based modification signatures at specific nucleosides are not the expected signal
- Input is raw FASTQ without a BAM alignment to a reference transcript — Mana analyses ONT alignments (.BAM), not unassembled reads
- Goal is general run/sample quality (read length N50, quality scores) rather than position-specific error signature — NanoPlot-style QC is the wrong lens here

## Inputs

- Sorted and indexed ONT direct RNA alignment BAM file (e.g. alignment.bam, produced via Minimap2 -ax map-ont and SAMtools)
- Reference transcript FASTA (e.g. file.fasta)
- Working directory mounted into the Mana container at /src

## Outputs

- Per-nucleotide error profile (mismatch/deletion/insertion counts relative to the reference transcript) in the out directory
- Read length distribution (full-length vs truncated molecule fractions)
- Mana mRNA QC report directory (e.g. /src/out)

## How to apply

Pull the pre-built Mana Docker image (self-building takes >= 20 minutes and is not recommended) and mount the working directory at /src with -v ${PWD}:/src. Run the mRNA analysis mode (python3 mana.py --mrna) supplying the dRNA nanopore alignment via -b and the reference transcript FASTA via -f, writing the QC report to -o; the analysis takes about 10 minutes. Mana internally relies on Pysam for BAM-level statistics, with per-nucleotide error profiles determined relative to the reference using pysamstats. Interpret the output error profile against the published signature for N1-methylpseudouridine: misclassification of modified sites as 0.62C/0.38U, total error rate of 23.6% at modified nucleosides (11.8% mismatch, 9% deletion, 2.8% insertion). Because the deletion enrichment shifts the read length profile, also read the length distribution report — expect roughly 41% full-length and 54% truncated molecules for the modified mRNA.

## Related tools

- **Mana** (Command-line tool run inside Docker (python3 mana.py --mrna) that analyses ONT alignments (.BAM) and emits the per-nucleotide error profile and mRNA length QC report) — github.com/scchess/Mana
- **Pysam** (Dependency used internally by Mana for BAM-level statistics on the aligned dRNA reads)
- **pysamstats** (Underlying utility (via Mana's dependency chain) for calculating per-genome-position variation statistics (matches, mismatches, deletions, insertions) from the SAM/BAM alignment) — https://github.com/alimanfoo/pysamstats

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Modified-site misclassification matches the reported signature 0.62C/0.38U
- Total error rate at modified nucleosides reproduces 23.6% (11.8% mismatch, 9% deletion, 2.8% insertion)
- Deletion-error enrichment shifts the mRNA length profile to ~41% full-length and ~54% truncated molecules
- Alignment input must be a sorted and indexed BAM (SAMtools-processed); Mana's out directory must contain both the per-nucleotide error profile and the read length distribution
- Run completes in roughly 10 minutes; failure to produce the out report suggests incorrect Docker volume mounting (-v ${PWD}:/src)

## Limitations

- Building the Mana Docker image from source takes >= 20 minutes and is not recommended; use the pre-built scchess/mana image
- Direct RNA sequencing is less reliable than cDNA sequencing; the authors anticipate dRNA mainly as a research tool, so error-based modification calling is not a manufacturing-grade assay
- The reported error signatures come from a proof-of-concept study with sample sizes ranging from n=1 to n=4

## Evidence

- [methods] Mana error signature workflow: [evidence span withheld — anchor status: not_found]
- [results] Published validation statistics: [evidence span withheld — anchor status: not_found]
- [discussion] pysamstats used for error profile: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [abstract] Mana tool description: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] Docker invocation: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta \
               -o /src/out"
- [readme] Docker rationale: "If you would like to build it yourself (>= 20 minutes and not recommended, a pre-built docker image is available)"
