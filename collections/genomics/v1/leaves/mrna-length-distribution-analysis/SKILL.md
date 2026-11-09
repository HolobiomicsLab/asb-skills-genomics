---
name: mrna-length-distribution-analysis
description: Use when analysing ONT nanopore alignments (.BAM) of synthetic mRNA vaccines
  to quantify full-length vs truncated read fractions, or when a nucleoside modification
  (N1-methylpseudouridine) is suspected to shift the mRNA length distribution via
  deletion errors.
when_to_use_negative:
- When the goal is sequence identity or contamination detection from cDNA sequencing
  rather than transcript length/integrity from direct RNA data
- When the alignment BAM has not been sorted and indexed (SAMtools v1.15 output) or
  lacks a matching reference transcript FASTA
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0080
tools:
- name: Mana
  role: Command-line tool that analyses ONT alignments (.BAM) via its --mrna mode
    to report mRNA QC statistics including read length distribution; internally relies
    on Pysam
  repo: https://github.com/scchess/Mana
- name: Pysam
  role: BAM-level statistics dependency used internally by Mana (v0.20.0)
  repo: https://pysam.readthedocs.io/en/latest/api.html
- name: pysamstats
  role: 'Computes the per-nucleotide error profile (variation statistics: matches,
    mismatches, deletions, insertions) against the reference that accompanies the
    length distribution'
  repo: https://github.com/alimanfoo/pysamstats
- name: SAMtools
  role: v1.15, used with 'SAMtools view -F 2048' to extract read length distributions
    of primary aligned reads (excluding secondary alignments)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:d27c8f728e5b6c7baf53106c6c641c3e889de0e3dc4141d03816f611139e205f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Pysam
  - SAMtools
  - pysamstats
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mrna-length-distribution-analysis

## Summary

Analysis of mRNA vaccine length distribution and integrity from nanopore sequencing alignments using the Mana toolkit, which reports the fraction of full-length versus truncated molecules and links per-nucleoside deletion errors (e.g. at N1-methylpseudouridine) to a shifted transcript length profile.

## When to use

Use when analysing ONT nanopore alignments (.BAM) of synthetic mRNA vaccines to quantify full-length vs truncated read fractions, or when a nucleoside modification (N1-methylpseudouridine) is suspected to shift the mRNA length distribution via deletion errors.

## When NOT to use

- When the goal is sequence identity or contamination detection from cDNA sequencing rather than transcript length/integrity from direct RNA data
- When the alignment BAM has not been sorted and indexed (SAMtools v1.15 output) or lacks a matching reference transcript FASTA

## Inputs

- Sorted, indexed ONT dRNA alignment BAM file (produced by Minimap2 -ax map-ont and SAMtools v1.15)
- Reference transcript FASTA file
- Optionally a primary-alignment-filtered read set (SAMtools view -F 2048) for read length distributions

## Outputs

- QC report directory containing the read length distribution with full-length vs truncated molecule fractions
- Per-nucleotide error profile (mismatch, deletion, insertion rates) relative to the reference transcript

## How to apply

Pull the pre-built Mana container with 'docker pull scchess/mana && docker tag scchess/mana mana' rather than building from source (which takes >= 20 minutes). Mount the working directory into the container with -v ${PWD}:/src and run the mRNA analysis mode: python3 mana.py --mrna, supplying the dRNA nanopore alignment via -b and the reference transcript FASTA via -f, directing the QC report to -o (analysis takes ~10 minutes). Mana internally relies on Pysam for BAM-level statistics; for read-length distributions of primary aligned reads specifically, the article extracts them with SAMtools view -F 2048 to exclude secondary alignments. Read length integrity should be interpreted jointly with the per-nucleotide error profile (computed with pysamstats against the reference): enrichment of deletion errors at modified nucleosides (9% deletions, part of a 23.6% total error rate at N1-methylpseudouridine sites) shifts the mRNA length profile toward truncated molecules.

## Related tools

- **Mana** (Command-line tool that analyses ONT alignments (.BAM) via its --mrna mode to report mRNA QC statistics including read length distribution; internally relies on Pysam) — https://github.com/scchess/Mana
- **Pysam** (BAM-level statistics dependency used internally by Mana (v0.20.0)) — https://pysam.readthedocs.io/en/latest/api.html
- **pysamstats** (Computes the per-nucleotide error profile (variation statistics: matches, mismatches, deletions, insertions) against the reference that accompanies the length distribution) — https://github.com/alimanfoo/pysamstats
- **SAMtools** (v1.15, used with 'SAMtools view -F 2048' to extract read length distributions of primary aligned reads (excluding secondary alignments))

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- For N1-methylpseudouridine-modified mRNA, length fractions should reproduce ~41% full-length and ~54% truncated molecules
- The per-nucleotide error profile at modified sites should total ~23.6% errors (11.8% mismatch, 9% deletion, 2.8% insertion), with deletion enrichment explaining the truncated-shifted length profile
- Misclassification of modified sites should match the reported 0.62C/0.38U pattern
- The out directory must contain both the per-nucleotide error profile and the read length distribution computed by Mana
- Read length distributions must be derived from primary aligned reads only (SAMtools view -F 2048), excluding secondary alignments

## Limitations

- Building the Mana container from source takes >= 20 minutes and is not recommended; use the pre-built docker image
- Analysis runtime is about 10 minutes per dataset
- Direct RNA sequencing is less reliable than cDNA sequencing per the authors, though still useful as a research tool for mRNA chemistry such as N1-methylpseudouridine detection
- Article data are proof of concept, ranging from n = 1 to n = 4, with no statistical method used to predetermine sample size

## Evidence

- [results] Direct RNA sequencing of N1-methylpseudouridine-modified mRNA showed an elevated error rate of 23.6% (11.8% mismatch, 9% deletion, 2.8% insertion) at modified nucleosides, and this enrichment in deletion errors shifted the mRNA length profile.: [evidence span withheld — anchor status: not_found]
- [readme] Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics.: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [methods] SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [methods] The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [abstract] VAX-seq can comprehensively measure key mRNA vaccine quality attributes, including sequence, length, integrity, and purity: "VAX-seq can comprehensively measure key mRNA vaccine quality attributes, including sequence, length, integrity, and purity"
