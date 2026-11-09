---
name: viral-consensus-genome-calling
description: Use when you have short-read alignments (unsorted SAM/BAM/CRAM) against
  a single reference sequence (e.g. SARS-CoV-2 NC_045512.2) and need a fast, memory-efficient
  consensus FASTA, especially inside a streaming pipeline.
when_to_use_negative:
- The reference is an ultra-long genome (e.g. a human chromosome), since memory consumption
  scales linearly with reference genome length and may be prohibitively large
- Consensus must be reconstructed from multiple reference sequences rather than a
  single reference sequence
- The dataset is very large and only a web browser environment is available — the
  WebAssembly web app works well for reasonably small datasets but larger datasets
  need the command-line tool
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: ViralConsensus
  role: 'Core consensus caller: streams alignments on-the-fly, tallies base/insertion
    counts per reference position, and writes the consensus FASTA'
  repo: https://github.com/niemasd/ViralConsensus
- name: htslib
  role: C library dependency used by ViralConsensus to read SAM/BAM/CRAM alignments
  repo: https://github.com/samtools/htslib
- name: minimap2
  role: Read mapper whose SAM output (-t 1 -a -x sr) is piped via standard input directly
    into viral_consensus in streaming mode
- name: samtools
  role: Used in the tee-based streaming variant to archive the mapper output as BAM
    (samtools view -b -@ 1) while consensus calling proceeds
- name: iVar
  role: Existing consensus pipeline used as the performance and accuracy baseline
    for benchmarking
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:e75077d7315c1e2a094e4f7e1be3d707fbfaace59beab26dd58eb21d6ca35d01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - C++
  - ViralConsensus
  - htslib
  - iVar
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# viral-consensus-genome-calling

## Summary

Call viral consensus genome sequences directly from unsorted SAM/BAM/CRAM read alignments using ViralConsensus, a C++/htslib tool that processes records on-the-fly with linear time and constant, low memory. It supports direct piping from read mappers such as minimap2 to avoid intermediary files and disk I/O.

## When to use

Use when you have short-read alignments (unsorted SAM/BAM/CRAM) against a single reference sequence (e.g. SARS-CoV-2 NC_045512.2) and need a fast, memory-efficient consensus FASTA, especially inside a streaming pipeline.

## When NOT to use

- The reference is an ultra-long genome (e.g. a human chromosome), since memory consumption scales linearly with reference genome length and may be prohibitively large
- Consensus must be reconstructed from multiple reference sequences rather than a single reference sequence
- The dataset is very large and only a web browser environment is available — the WebAssembly web app works well for reasonably small datasets but larger datasets need the command-line tool

## Inputs

- Unsorted or sorted read alignment file in SAM, BAM, or CRAM format (path or '-' for standard input)
- Single-sequence reference genome FASTA (e.g. SARS-CoV-2 NC_045512.2.fas)
- Optional FASTQ reads and mapper (minimap2 -a -x sr) when using the piped streaming mode

## Outputs

- Consensus genome sequence in FASTA format (e.g. consensus.fas)
- Optionally, benchmark tables of wall-clock runtime and peak RSS per tool/replicate (benchmark_results.csv/.tsv)

## How to apply

Build ViralConsensus from source after installing htslib (autoreconf -i; ./configure; make; make install), setting LD_LIBRARY_PATH to include /usr/local/lib, then run `viral_consensus -i <alignment> -r <reference.fas> -o <consensus.fas>`; input may be an unsorted BAM (sorting is not required because base and insertion counts are tallied in memory as records stream by). For pipeline integration, pipe the mapper output directly via standard input (`-i -`), e.g. minimap2 with `-t 1 -a -x sr` for short reads, optionally using tee to also archive the stream as BAM via `samtools view -b -@ 1`. When benchmarking against alternatives like the iVar pipeline, profile wall-clock runtime and peak RSS with /usr/bin/time -v across subsampled depths (e.g. 100 to 1M mapped reads with replicates). Validate correctness by comparing the emitted consensus FASTA against a reference implementation on identical inputs, and confirm performance targets such as ~1M reads in under 2 seconds with under 12 MB peak memory.

## Related tools

- **ViralConsensus** (Core consensus caller: streams alignments on-the-fly, tallies base/insertion counts per reference position, and writes the consensus FASTA) — https://github.com/niemasd/ViralConsensus
- **htslib** (C library dependency used by ViralConsensus to read SAM/BAM/CRAM alignments) — https://github.com/samtools/htslib
- **minimap2** (Read mapper whose SAM output (-t 1 -a -x sr) is piped via standard input directly into viral_consensus in streaming mode)
- **samtools** (Used in the tee-based streaming variant to archive the mapper output as BAM (samtools view -b -@ 1) while consensus calling proceeds)
- **iVar** (Existing consensus pipeline used as the performance and accuracy baseline for benchmarking)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Consensus FASTA produced from the example/example.trimmed.unsorted.bam and NC_045512.2.fas inputs matches the consensus emitted by viral_consensus v0.0.1 on identical inputs
- Runtime scales linearly with mapped-read count while peak memory stays constant as depth increases (100 to 1M reads)
- 1M-read SARS-CoV-2 amplicon sample completes in under 2 seconds with peak RSS under 12 MB
- Piped mode (mapper to stdin) writes fewer bytes to disk than the two-step mode that writes an intermediary SAM file, while producing an identically formatted consensus FASTA
- Normalized Hamming error of the reconstructed consensus versus its source reference genome is comparable to iVar v1.3.1 across lineages, coverages (30x/40x/50x), and sequencing technologies

## Limitations

- Memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory
- Requires a single reference sequence being used for reconstruction; it is primarily meant for viral genomics although usable for bacterial or mitochondrial genomes
- The WebAssembly web app works well only for reasonably small datasets; larger datasets require the command-line tool
- No discussion-section changelog was present in the provided document

## Evidence

- [abstract] Streaming consensus calling without sorting: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [abstract] Speed and memory efficiency claim: "ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods."
- [methods] Implementation dependencies: "ViralConsensus is written in C++ and depends on htslib"
- [methods] Basic CLI invocation: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [methods] Piped mapper invocation: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [readme] Memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [readme] Installation environment variable: "export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib""
