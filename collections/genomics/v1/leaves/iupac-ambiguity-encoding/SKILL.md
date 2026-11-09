---
name: iupac-ambiguity-encoding
description: Use when deriving a majority-rule consensus from MinION read-to-target
  alignments (e.g., LAST MAF output) and ambiguous positions must be represented with
  IUPAC nomenclature rather than forced single-base calls.
when_to_use_negative:
- The goal is high-confidence variant/SNP calling rather than consensus display —
  NanoPipe's own discussion notes its SNP detection statistics are not yet sophisticated
  and more sophisticated statistics are planned future work.
- The analysis is metagenomics — NanoPipe does not yet support metagenomics analysis;
  implementation is planned.
- Positions lack read data entirely and a de novo assembly consensus (rather than
  a target-guided consensus from alignments) is required.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: NanoPipe
  role: Web-driven automatic pipeline whose alignment-analysis stage aggregates per-position
    nucleotide counts from the LAST alignments and emits the IUPAC-encoded consensus
    per target in FASTA
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) producing the read-to-target MAF alignments
    that serve as the input to the consensus calculation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:5c827f4a26223e6b0786bb83e737d2e90c160638979949d4f1b6b00788dad610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# iupac-ambiguity-encoding

## Summary

Generates a per-target consensus sequence from nanopore read-to-target alignments by majority-rule nucleotide assignment, encoding positions where the top two nucleotide counts differ by ≤20% with IUPAC ambiguity codes and gapping positions with fewer than 10 supporting reads.

## When to use

Use when deriving a majority-rule consensus from MinION read-to-target alignments (e.g., LAST MAF output) and ambiguous positions must be represented with IUPAC nomenclature rather than forced single-base calls.

## When NOT to use

- The goal is high-confidence variant/SNP calling rather than consensus display — NanoPipe's own discussion notes its SNP detection statistics are not yet sophisticated and more sophisticated statistics are planned future work.
- The analysis is metagenomics — NanoPipe does not yet support metagenomics analysis; implementation is planned.
- Positions lack read data entirely and a de novo assembly consensus (rather than a target-guided consensus from alignments) is required.

## Inputs

- read-to-target alignments in MAF format from LAST version 946 (MinION reads mapped to target chromosome/region)
- MinION sequencing reads in FASTA or FASTQ format (upstream alignment input)

## Outputs

- individual consensus sequence per target chromosome/region in FASTA format
- gaps at consensus positions supported by fewer than 10 reads
- IUPAC ambiguity codes at positions where the top two nucleotide counts differ by at most 20%

## How to apply

Parse the read-to-target MAF alignments produced by LAST version 946 in the NanoPipe alignment-analysis stage and aggregate nucleotide counts at each position of each target chromosome/region across all mapped reads. Require a minimum nucleotide count of 10 reads per position; below this threshold assign a gap. Assign the majority-rule nucleotide (the nucleotide with the higher count) to each consensus position. Where the counts of any two nucleotides differ from each other by not more than 20%, include both nucleotides by encoding the position with the corresponding IUPAC ambiguity code instead of forcing a single base. Write the resulting per-target consensus sequence to the consensus output file in FASTA format for viewing and download.

## Related tools

- **NanoPipe** (Web-driven automatic pipeline whose alignment-analysis stage aggregates per-position nucleotide counts from the LAST alignments and emits the IUPAC-encoded consensus per target in FASTA) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) producing the read-to-target MAF alignments that serve as the input to the consensus calculation)

## Examples

```
bash NANOPIPE/check.sh
```

## Evaluation signals

- Every consensus position with fewer than 10 supporting reads is encoded as a gap.
- Positions where the counts of any two nucleotides differ by not more than 20% carry the correct IUPAC ambiguity code rather than a single nucleotide.
- All other positions carry the majority-rule nucleotide (the nucleotide with the higher count).
- A separate consensus entry is emitted per target chromosome/region, in FASTA format, matching the alignment-analysis stage's per-target grouping.
- In the Plasmodium case study, the pipeline mapped 99.9% of all query reads and detected the expected mutations, so the consensus should reflect those known mutations.

## Limitations

- Positions with fewer than 10 reads become gaps, so low-coverage regions yield uninformative consensus rather than calls.
- SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work, so ambiguity encoding should not be over-interpreted as variant calling.
- NanoPipe was built for MinION data specifically; alignment parameters are adjusted for MinION sequencing technology specifics and may not suit other platforms.

## Evidence

- [methods] majority rule: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] 20% IUPAC rule: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [methods] minimum count 10: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [abstract] consensus in results: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [readme] README scope: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest"
