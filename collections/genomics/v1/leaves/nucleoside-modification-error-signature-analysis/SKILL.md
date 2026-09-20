---
name: nucleoside-modification-error-signature-analysis
description: Use when direct RNA (dRNA) nanopore alignments (.BAM) of synthetic mRNA
  are available and the goal is to detect N1-methylpseudouridine or similar nucleoside
  modifications via characteristic error signatures and altered transcript length
  profiles.
when_to_use_negative:
- The input is cDNA or Illumina sequencing data rather than direct RNA nanopore alignments
  — the article found cDNA sequencing more reliable and this error-signature approach
  depends on dRNA chemistry readout.
- The goal is generic mRNA QC (sequence identity, integrity, poly(A) tail length,
  contamination) without needing nucleoside-modification detection — use the broader
  VAX-seq/Mana quality workflow instead.
- The alignment BAM is absent or not sorted/indexed from ONT direct RNA reads; Mana
  analyses ONT alignments and cannot infer modifications from raw FASTQ.
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0080
tools:
- name: Mana
  role: Command-line toolkit that analyses ONT alignments (.BAM) in mRNA mode to compute
    the per-nucleotide error profile and read length distribution used as the modification
    signature
  repo: https://github.com/scchess/Mana
- name: Pysam
  role: Dependency used internally by Mana for BAM-level statistics underlying the
    error profile
  repo: https://pysam.readthedocs.io/en/latest/api.html
- name: pysamstats
  role: Utility for calculating per-nucleotide statistics (matches, mismatches, deletions,
    insertions) against genome positions from BAM alignments
  repo: https://github.com/alimanfoo/pysamstats
- name: SAMtools v1.15
  role: Generates the sorted and indexed BAM files that feed Mana's modification-signature
    analysis
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT reads to the reference with -ax map-ont prior to error-signature
    extraction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:84e5fc439027cb41436181465f5ad16e3733b24058f320a9f83132a05c132427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# nucleoside-modification-error-signature-analysis

## Summary

Detects RNA nucleoside modifications such as N1-methylpseudouridine from direct RNA nanopore sequencing by quantifying per-nucleoside error signatures (mismatches, deletions, insertions) and the resulting read length distribution, using the Mana toolkit over ONT BAM alignments.

## When to use

Use when direct RNA (dRNA) nanopore alignments (.BAM) of synthetic mRNA are available and the goal is to detect N1-methylpseudouridine or similar nucleoside modifications via characteristic error signatures and altered transcript length profiles.

## When NOT to use

- The input is cDNA or Illumina sequencing data rather than direct RNA nanopore alignments — the article found cDNA sequencing more reliable and this error-signature approach depends on dRNA chemistry readout.
- The goal is generic mRNA QC (sequence identity, integrity, poly(A) tail length, contamination) without needing nucleoside-modification detection — use the broader VAX-seq/Mana quality workflow instead.
- The alignment BAM is absent or not sorted/indexed from ONT direct RNA reads; Mana analyses ONT alignments and cannot infer modifications from raw FASTQ.

## Inputs

- dRNA nanopore alignment file (.BAM) of reads mapped to the mRNA construct
- Reference transcript sequence (.fasta)
- Pre-built Mana Docker image (scchess/mana)

## Outputs

- Per-nucleotide error profile (mismatch, deletion, insertion rates at modified nucleosides)
- Read length distribution (full-length vs truncated mRNA fractions)
- QC report directory (e.g. /src/out) with mRNA quality statistics

## How to apply

Pull the pre-built Mana Docker image (`docker pull scchess/mana && docker tag scchess/mana mana`) rather than self-building (>= 20 min). Mount the working directory at /src with `-v ${PWD}:/src` and run the mRNA analysis mode (`python3 mana.py --mrna`) supplying the dRNA nanopore alignment via `-b /src/alignment.bam` and the reference transcript via `-f /src/file.fasta`, writing QC reports to `-o /src/out`; the run takes about 10 minutes and Mana internally relies on Pysam for BAM-level statistics. The rationale is that N1-methylpseudouridine disrupts nanopore basecalling, so modified nucleosides accumulate elevated error rates and the enrichment of deletion errors shifts the mRNA length profile. Validate by comparing the per-nucleotide error profile and read length distribution against reported reference values.

## Related tools

- **Mana** (Command-line toolkit that analyses ONT alignments (.BAM) in mRNA mode to compute the per-nucleotide error profile and read length distribution used as the modification signature) — https://github.com/scchess/Mana
- **Pysam** (Dependency used internally by Mana for BAM-level statistics underlying the error profile) — https://pysam.readthedocs.io/en/latest/api.html
- **pysamstats** (Utility for calculating per-nucleotide statistics (matches, mismatches, deletions, insertions) against genome positions from BAM alignments) — https://github.com/alimanfoo/pysamstats
- **SAMtools v1.15** (Generates the sorted and indexed BAM files that feed Mana's modification-signature analysis)
- **Minimap2 (Release 2.20-r1064)** (Aligns ONT reads to the reference with -ax map-ont prior to error-signature extraction)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Misclassification of modified N1-methylpseudouridine sites reproduces 0.62C / 0.38U call fractions
- Total error rate at modified nucleosides reaches ~23.6%, decomposing into 11.8% mismatch, 9% deletion, and 2.8% insertion
- Length distribution shows ~41% full-length and ~54% truncated molecules, reflecting the deletion-error enrichment
- Mana run completes (~10 minutes) and writes the QC report to the directory given with -o; the out directory contains the per-nucleotide error profile and read length distribution
- Input BAM is sorted and indexed (SAMtools v1.15) so Mana/Pysam can traverse alignments correctly

## Limitations

- The pre-built Mana Docker image should be used; self-building takes >= 20 minutes and is not recommended
- Data in the source article are proof of concept with small sample sizes, ranging from n = 1 to n = 4
- cDNA sequencing was found more reliable than direct RNA sequencing, so modification detection via dRNA errors is positioned as a research tool rather than the most reliable readout
- Experiments were not randomised and investigators were not blinded; template quality is the most significant determinant of sequencing quality

## Evidence

- [results] dRNA error signature: [evidence span withheld — anchor status: not_found]
- [readme] Mana workflow: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta \
               -o /src/out"
- [abstract] Mana purpose: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [results] length distribution shift: [evidence span withheld — anchor status: not_found]
- [discussion] per-nucleotide profile basis: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
