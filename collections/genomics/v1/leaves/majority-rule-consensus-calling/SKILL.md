---
name: majority-rule-consensus-calling
description: Use when nanopore MinION reads have been mapped to target sequences and
  a reliable FASTA consensus per target chromosome/region is needed from the alignments.
when_to_use_negative:
- The goal is calling individual polymorphisms/SNPs rather than a consensus sequence
  (SNP candidates use separate coverage thresholds of ≥20% of reads and ≥30% of maximum
  contig coverage)
- Read-to-target alignments do not exist yet — alignment with LAST must be run first
  via NanoPipe's alignment stage
- Metagenomics analysis is intended, which NanoPipe does not yet support
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: NanoPipe
  role: Web-driven automatic pipeline whose alignment-analysis and results-display
    stages compute and emit the per-target consensus sequence
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) producing the read-to-target MAF alignments
    from which nucleotide counts per position are aggregated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:3f063a66c25611076aecde3f571e4a382b52cb6dad4242c36bb140bd4ac51b2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# majority-rule-consensus-calling

## Summary

Generates a per-chromosome/region consensus DNA sequence from read-to-target alignments (LAST v946 MAF files produced by NanoPipe) using majority-rule nucleotide assignment with a minimum 10-read coverage and IUPAC ambiguity encoding when top two nucleotide counts differ by ≤20%.

## When to use

Use when nanopore MinION reads have been mapped to target sequences and a reliable FASTA consensus per target chromosome/region is needed from the alignments.

## When NOT to use

- The goal is calling individual polymorphisms/SNPs rather than a consensus sequence (SNP candidates use separate coverage thresholds of ≥20% of reads and ≥30% of maximum contig coverage)
- Read-to-target alignments do not exist yet — alignment with LAST must be run first via NanoPipe's alignment stage
- Metagenomics analysis is intended, which NanoPipe does not yet support

## Inputs

- Read-to-target alignments in MAF format from LAST version 946 (NanoPipe alignment stage)
- MinION sequencing reads in FASTA or FASTQ format (uploaded to NanoPipe)
- Target reference sequences (target database)

## Outputs

- Consensus sequence in FASTA format, one per target chromosome/region
- Gap characters at positions supported by fewer than 10 reads
- IUPAC ambiguity codes where top two nucleotide counts differ by ≤20%

## How to apply

Parse the read-to-target MAF alignments produced by LAST version 946 via NanoPipe's alignment-analysis stage. For each target position, aggregate nucleotide counts across all mapped reads and apply majority rule: assign the nucleotide with the higher count. Assign a gap wherever the minimum nucleotide count per position falls below 10 reads. If the counts for any two nucleotides differ by not more than 20%, include both nucleotides encoded in IUPAC nomenclature. Write one consensus per target chromosome/region to a FASTA output file, then validate that gaps and IUPAC codes appear exactly where the count thresholds dictate.

## Related tools

- **NanoPipe** (Web-driven automatic pipeline whose alignment-analysis and results-display stages compute and emit the per-target consensus sequence) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) producing the read-to-target MAF alignments from which nucleotide counts per position are aggregated)

## Examples

```
bash NANOPIPE/check.sh
```

## Evaluation signals

- Every consensus position where nucleotide count is below 10 reads is encoded as a gap
- Positions where the top two nucleotide counts differ by not more than 20% carry the correct IUPAC ambiguity code
- Output file is valid FASTA, with one individual consensus emitted per target chromosome/region
- Majority-rule nucleotide at each position matches the nucleotide with the higher count among mapped reads
- In the Plasmodium falciparum case study, the pipeline mapped 99.9% of query reads and detected the expected mutations, giving confidence in alignment inputs

## Limitations

- Positions with fewer than 10 supporting reads yield gaps, so low-coverage target regions produce incomplete consensus
- Ambiguity encoding only triggers when top two nucleotide counts differ by ≤20%; more complex mixtures are not represented in the consensus
- NanoPipe's SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- Metagenomics analysis is not yet supported

## Evidence

- [methods] The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [abstract] The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
