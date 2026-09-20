---
name: per-position-base-insertion-count-tallying
description: Use when consensus sequences must be derived directly from read alignments
  (SAM/BAM/CRAM, possibly unsorted or streamed from a mapper) and you need fast, memory-efficient
  per-position base and insertion tallies rather than sorted pileups.
when_to_use_negative:
- Ultra-long reference genomes (e.g. human chromosomes), since memory consumption
  scales linearly with reference genome length and may require prohibitively large
  amounts of memory
- Workflows that require sorted, indexed BAM pileups or per-read variant evidence
  rather than a single most-frequent-base consensus
edam_operation: http://edamontology.org/operation_3483
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ViralConsensus
  role: C++/htslib tool that streams unsorted alignments on-the-fly, tallies per-position
    base and insertion counts, and emits the FASTA consensus
  repo: https://github.com/niemasd/ViralConsensus
- name: htslib
  role: Core C library ViralConsensus depends on for reading SAM/BAM/CRAM alignment
    records
  repo: https://github.com/samtools/htslib
- name: minimap2
  role: Read mapper whose SAM output is piped via standard input into the tallying
    step for on-the-fly consensus calling
- name: samtools
  role: Used with tee to archive the streamed mapper output as BAM (samtools view
    -b -@ 1) while the tallies are computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:a60a6415fe2eacfec0a8dc1216b1ae31f830cd7025cade20ae769acaf408b0b4
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
  edam_operation: http://edamontology.org/operation_3483
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-position base-insertion count tallying

## Summary

Stream unsorted SAM/BAM/CRAM read alignments one record at a time and increment in-memory base counts at each reference-genome position and insertion counts before each position, so consensus calling can happen on-the-fly without sorting or intermediate pileup files.

## When to use

Use when consensus sequences must be derived directly from read alignments (SAM/BAM/CRAM, possibly unsorted or streamed from a mapper) and you need fast, memory-efficient per-position base and insertion tallies rather than sorted pileups.

## When NOT to use

- Ultra-long reference genomes (e.g. human chromosomes), since memory consumption scales linearly with reference genome length and may require prohibitively large amounts of memory
- Workflows that require sorted, indexed BAM pileups or per-read variant evidence rather than a single most-frequent-base consensus

## Inputs

- Unsorted read alignments in SAM/BAM/CRAM format (file path or '-' for standard input)
- Single reference sequence in FASTA format (e.g. example/NC_045512.2.fas)
- User-provided base quality threshold

## Outputs

- Consensus genome sequence in FASTA format (e.g. consensus.fas)
- In-memory per-position base counts and pre-position insertion counts

## How to apply

Write or use a C++ program linked against htslib that accepts alignments via -i (file path or '-' for standard input), a single reference sequence via -r, and an output path via -o. Stream records one at a time without requiring coordinate sorting; for each column of the alignment, if the user-provided base quality threshold is met, increment the base count at the corresponding reference position and the insertion count before that position, both held in memory. After streaming, emit the most frequent base at each reference position into the consensus in FASTA format. To avoid disk I/O, pipe mapper output directly (minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz) into the tallying tool, using tee with samtools view -b -@ 1 to simultaneously archive the BAM.

## Related tools

- **ViralConsensus** (C++/htslib tool that streams unsorted alignments on-the-fly, tallies per-position base and insertion counts, and emits the FASTA consensus) — https://github.com/niemasd/ViralConsensus
- **htslib** (Core C library ViralConsensus depends on for reading SAM/BAM/CRAM alignment records) — https://github.com/samtools/htslib
- **minimap2** (Read mapper whose SAM output is piped via standard input into the tallying step for on-the-fly consensus calling)
- **samtools** (Used with tee to archive the streamed mapper output as BAM (samtools view -b -@ 1) while the tallies are computed)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Run on example/example.trimmed.unsorted.bam with example/NC_045512.2.fas and confirm the emitted consensus.fas matches the consensus produced by viral_consensus v0.0.1 on identical inputs
- Verify the implementation processes the unsorted BAM directly without any sorting step or coordinate-sorted index
- Confirm output is a single FASTA sequence whose length matches the reference sequence
- When invoked via standard input (-i -), verify the piped mapper stream is consumed without writing intermediate alignment files to disk

## Limitations

- Memory consumption scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may require prohibitively large amounts of memory
- Requires a single reference sequence being used for reconstruction; not applicable when aligning to multiple references
- The web app (WebAssembly port) works well only for reasonably small datasets; larger datasets require the command-line tool

## Evidence

- [methods] streaming on-the-fly without sorting: [evidence span withheld — anchor status: not_found]
- [methods] base quality threshold and count increments: "if the user-provided base quality threshold is met, the base/insertion count at the corresponding position of the reference genome is incremented"
- [abstract] piping from read mapper: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] example command: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [readme] memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
