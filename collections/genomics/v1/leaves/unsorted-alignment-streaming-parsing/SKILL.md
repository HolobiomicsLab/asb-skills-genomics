---
name: unsorted-alignment-streaming-parsing
description: Use when you have unsorted read alignments (SAM/BAM/CRAM) against a single
  reference sequence and need a fast, memory-efficient consensus genome, especially
  when piping directly from a read mapper like minimap2.
when_to_use_negative:
- The reference genome is ultra-long (e.g. a human chromosome), since memory consumption
  scales linearly with reference genome length and may be prohibitively large
- The task requires coordinate-sorted, indexed BAM access or per-variant calls rather
  than a single per-position consensus from a single reference sequence
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0080
tools:
- name: ViralConsensus
  role: 'Reference implementation: streams unsorted alignments on-the-fly and calls
    the consensus genome'
  repo: https://github.com/niemasd/ViralConsensus
- name: htslib
  role: C library used to read SAM/BAM/CRAM alignment records in the streaming parser
  repo: https://github.com/samtools/htslib
- name: minimap2
  role: Read mapper whose output is piped via standard input directly into the consensus
    caller
- name: samtools
  role: Used with tee to archive the mapper stream as BAM (samtools view -b) while
    consensus calling proceeds
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:e03ce0a70b4abae9855868626a0365542ed68e95e79e042e9393d8464e146f4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - C++
  - ViralConsensus
  - htslib
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# unsorted-alignment-streaming-parsing

## Summary

A method for calling viral consensus genomes by streaming unsorted SAM/BAM/CRAM alignments on-the-fly with htslib, incrementing base and insertion counts per reference position instead of sorting or loading full pileups, as implemented in ViralConsensus.

## When to use

Use when you have unsorted read alignments (SAM/BAM/CRAM) against a single reference sequence and need a fast, memory-efficient consensus genome, especially when piping directly from a read mapper like minimap2.

## When NOT to use

- The reference genome is ultra-long (e.g. a human chromosome), since memory consumption scales linearly with reference genome length and may be prohibitively large
- The task requires coordinate-sorted, indexed BAM access or per-variant calls rather than a single per-position consensus from a single reference sequence

## Inputs

- Unsorted SAM/BAM/CRAM read alignment file or standard input stream from a read mapper
- FASTA reference sequence (single reference sequence, e.g. NC_045512.2.fas)
- User-provided base quality threshold

## Outputs

- Consensus genome sequence in FASTA format (consensus.fas)

## How to apply

Write or invoke a C++ program linked against htslib that accepts unsorted alignments via -i (file path or '-' for stdin), a single reference sequence via -r, and an output path via -o. Stream alignment records one at a time without coordinate sorting; for each column of the alignment, if the user-provided base quality threshold is met, increment the base count at the corresponding reference position and insertion counts before every position held in memory. After streaming, emit the most frequent base at each reference position into a FASTA consensus. To avoid disk I/O, pipe mapper output directly (e.g. minimap2 -a -x sr output via stdin), using tee with samtools view -b if the BAM should also be archived. Validate by confirming the output consensus matches the one produced by viral_consensus v0.0.1 on identical inputs (e.g. example.trimmed.unsorted.bam with NC_045512.2 reference).

## Related tools

- **ViralConsensus** (Reference implementation: streams unsorted alignments on-the-fly and calls the consensus genome) — https://github.com/niemasd/ViralConsensus
- **htslib** (C library used to read SAM/BAM/CRAM alignment records in the streaming parser) — https://github.com/samtools/htslib
- **minimap2** (Read mapper whose output is piped via standard input directly into the consensus caller)
- **samtools** (Used with tee to archive the mapper stream as BAM (samtools view -b) while consensus calling proceeds)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Output FASTA consensus matches the consensus produced by viral_consensus v0.0.1 on the same trimmed unsorted BAM and NC_045512.2 reference
- The tool runs successfully on an unsorted BAM without any prior coordinate sorting or index-building step
- Consensus is produced when reading directly from standard input ('-i -') fed by a piped read mapper, confirming on-the-fly operation
- Every reference position in the output FASTA corresponds to the most frequent base tallied at that position (with insertions tallied before each position)

## Limitations

- Memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large memory
- Requires a single reference sequence being used for reconstruction; multiple-reference scenarios are out of scope
- WebAssembly web app port works well only for reasonably small datasets; larger datasets need the command-line tool

## Evidence

- [other] Read alignments are streamed on-the-fly without any need for sorting: [evidence span withheld — anchor status: not_found]
- [methods] ViralConsensus is written in C++ and depends on htslib: "ViralConsensus is written in C++ and depends on htslib"
- [readme] memory consumption scales linearly with reference genome length: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] data can be piped directly from the read mapper: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
