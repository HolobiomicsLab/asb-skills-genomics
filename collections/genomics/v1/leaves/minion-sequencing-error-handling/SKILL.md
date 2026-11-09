---
name: minion-sequencing-error-handling
description: Use when raw MinION/ONT sequencing reads (FASTA/FASTQ) have been mapped
  to a target reference and a per-target consensus sequence must be computed while
  handling nanopore error via read-count and IUPAC ambiguity thresholds.
when_to_use_negative:
- Data are not from Oxford Nanopore MinION devices or were not aligned to a target
  with LAST — the consensus thresholds are tuned to MinION error specifics
- Metagenomics analysis is required — NanoPipe does not yet support it (implementation
  planned)
- Sophisticated SNP detection/evaluation statistics are required — current SNP detection
  statistics are basic and are planned future work
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPipe
  role: Web-driven pipeline that performs the alignment analysis and majority-rule
    consensus calculation per target chromosome/region
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Version 946 sequence aligner producing the read-to-target MAF alignments from
    which nucleotide counts are aggregated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:95b2b4b874133acbcae60d3e0b0cf5cc57c0ef033cdbe4396cff1ec601911a18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# minion-sequencing-error-handling

## Summary

Generates a majority-rule consensus sequence per target chromosome/region from MinION read-to-target alignments (LAST v946 MAF) as implemented in NanoPipe, using coverage and ambiguity thresholds to control nanopore sequencing error.

## When to use

Use when raw MinION/ONT sequencing reads (FASTA/FASTQ) have been mapped to a target reference and a per-target consensus sequence must be computed while handling nanopore error via read-count and IUPAC ambiguity thresholds.

## When NOT to use

- Data are not from Oxford Nanopore MinION devices or were not aligned to a target with LAST — the consensus thresholds are tuned to MinION error specifics
- Metagenomics analysis is required — NanoPipe does not yet support it (implementation planned)
- Sophisticated SNP detection/evaluation statistics are required — current SNP detection statistics are basic and are planned future work

## Inputs

- MinION sequencing reads in FASTA or FASTQ format
- Target reference sequences (e.g., P. falciparum 3D7, hg38, E. coli GCF_000005845.2)
- Read-to-target alignments in MAF format from LAST version 946

## Outputs

- Consensus sequence per target chromosome/region in FASTA format
- Gap characters at positions supported by fewer than 10 reads
- IUPAC ambiguity codes at positions where top-two nucleotide counts differ by at most 20%

## How to apply

Parse the read-to-target alignments produced by LAST version 946 in NanoPipe's alignment-analysis stage, then aggregate nucleotide counts at each position of the target across all mapped reads. Assign the majority-rule nucleotide (the nucleotide with the higher count) to each consensus position, and assign a gap where the minimum nucleotide count per position falls below 10 reads (i.e., fewer than 10 reads support the position). Where the counts for any two nucleotides differ from each other by not more than 20%, include both nucleotides in the consensus using IUPAC nomenclature instead of forcing a single base. Write the per-chromosome/region consensus to an output file in FASTA format, which the user can view and download; NanoPipe also emits polymorphism data alongside the consensus.

## Related tools

- **NanoPipe** (Web-driven pipeline that performs the alignment analysis and majority-rule consensus calculation per target chromosome/region) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Version 946 sequence aligner producing the read-to-target MAF alignments from which nucleotide counts are aggregated)

## Examples

```
bash NANOPIPE/check.sh  # verify installed packages (last, samtools) before running the NanoPipe consensus stage locally
```

## Evaluation signals

- Emitted consensus FASTA contains gap characters exactly at positions supported by fewer than 10 reads
- Positions where the top two nucleotide counts differ by at most 20% carry IUPAC ambiguity codes rather than a single base
- Consensus is emitted as an individual sequence per target chromosome/region, not a single merged contig
- Validation in the Plasmodium study case: 99.9% of query reads mapped and expected mutations were detected
- A full 21 MB FASTA query against the P. falciparum genome should complete in less than 4 minutes on NanoPipe

## Limitations

- Consensus positions with fewer than 10 supporting reads are assigned gaps, so low-coverage targets yield gapped consensus sequences
- SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- Metagenomics analysis is not supported; implementation is planned
- Too-short reads can create mapping noise unless the user sets a read length limit (e.g., 200 nucleotides)

## Evidence

- [methods] minimum count of 10 reads per position: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] majority rule with IUPAC at 20%: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] IUPAC inclusion threshold: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [abstract] results contents: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [readme] README scope: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
