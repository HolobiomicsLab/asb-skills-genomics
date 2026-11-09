---
name: htslib-c-plus-plus-alignment-processing
description: Use when you need to call a consensus genome directly from unsorted read
  alignments (SAM/BAM/CRAM) against a single reference sequence, or pipe mapper output
  on-the-fly without sorting or intermediate disk I/O.
when_to_use_negative:
- 'The reference genome is ultra-long (e.g. a human chromosome): memory consumption
  scales linearly with reference genome length and may require prohibitively large
  amounts of memory'
- The alignment spans multiple reference sequences — the tool requires a single reference
  sequence being used for reconstruction
- You need a small, browser-based run for a reasonably small dataset — the WebAssembly
  web app may suffice instead of the command-line tool
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: htslib
  role: C library linked from C++ to read and stream SAM/BAM/CRAM alignment records
  repo: https://github.com/samtools/htslib
- name: ViralConsensus
  role: Reference implementation whose on-the-fly streaming consensus-calling loop
    this skill reconstructs
  repo: https://github.com/niemasd/ViralConsensus
- name: minimap2
  role: Read mapper whose output is piped via standard input into the consensus caller
    for on-the-fly calling
- name: samtools
  role: Used as 'samtools view -b -@ 1' with tee to archive the mapper stream as BAM
    while consensus calling proceeds
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:a9c52dfe684d13c600a6be38414dad38950400edf4e4dfc90adb83cb0e814c67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samtools/htslib
  tools:
  - C++
  - ViralConsensus
  - htslib
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# htslib-c-plus-plus-alignment-processing

## Summary

Write a C++ program linked against htslib that streams unsorted SAM/BAM/CRAM read alignments on-the-fly and tallies base and insertion counts per reference position to call a consensus genome, as done in ViralConsensus.

## When to use

Use when you need to call a consensus genome directly from unsorted read alignments (SAM/BAM/CRAM) against a single reference sequence, or pipe mapper output on-the-fly without sorting or intermediate disk I/O.

## When NOT to use

- The reference genome is ultra-long (e.g. a human chromosome): memory consumption scales linearly with reference genome length and may require prohibitively large amounts of memory
- The alignment spans multiple reference sequences — the tool requires a single reference sequence being used for reconstruction
- You need a small, browser-based run for a reasonably small dataset — the WebAssembly web app may suffice instead of the command-line tool

## Inputs

- Unsorted read alignments in SAM, BAM, or CRAM format (file path or '-' for standard input)
- Reference genome in FASTA format (single reference sequence)
- User-provided base quality threshold
- Optionally, raw reads (e.g. FASTQ) streamed through a read mapper such as minimap2

## Outputs

- Consensus genome sequence in FASTA format (e.g. consensus.fas)

## How to apply

Implement a C++ tool linked against htslib that accepts alignments via -i (file path or '-' for standard input), a single reference sequence via -r, and an output path via -o. Stream alignment records one at a time without requiring coordinate sorting; for each column of the alignment, if the user-provided base quality threshold is met, increment the base count at the corresponding reference position and the insertion count before that position, both held in memory. After streaming, emit the most frequent base at each reference position into the consensus and write it in FASTA format. Support direct piping from a read mapper such as minimap2 (e.g. 'minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz') so consensus calling happens on-the-fly, using tee with 'samtools view -b -@ 1' to archive the mapper output as BAM in parallel.

## Related tools

- **htslib** (C library linked from C++ to read and stream SAM/BAM/CRAM alignment records) — https://github.com/samtools/htslib
- **ViralConsensus** (Reference implementation whose on-the-fly streaming consensus-calling loop this skill reconstructs) — https://github.com/niemasd/ViralConsensus
- **minimap2** (Read mapper whose output is piped via standard input into the consensus caller for on-the-fly calling)
- **samtools** (Used as 'samtools view -b -@ 1' with tee to archive the mapper stream as BAM while consensus calling proceeds)

## Examples

```
viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas
```

## Evaluation signals

- Run the implementation on the example.trimmed.unsorted.bam and NC_045512.2.fas reference and confirm the emitted consensus.fas matches the consensus produced by viral_consensus v0.0.1 on identical inputs
- Verify the tool accepts unsorted (non-coordinate-sorted) BAM input without error and produces the same consensus as with sorted input
- Verify piping works: 'minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | <implementation> -i -' completes without writing intermediate SAM/BAM to disk
- Check that FASTA output contains exactly one sequence with the same length and naming as the single reference sequence

## Limitations

- Memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory
- Requires a single reference sequence being used for reconstruction; cannot handle multi-sequence reference assemblies
- The web app (WebAssembly port) works well only for reasonably small datasets; larger datasets need the command-line tool

## Evidence

- [methods] streaming on-the-fly without sorting: [evidence span withheld — anchor status: not_found]
- [abstract] piping support: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [readme] implementation language and dependency: "ViralConsensus is written in C++ and depends on htslib"
- [readme] memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] tee archiving pattern: "`tee` should be used instead to send the stream down two (or more) paths"
