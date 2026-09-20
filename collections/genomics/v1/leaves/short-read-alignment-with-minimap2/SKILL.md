---
name: short-read-alignment-with-minimap2
description: Use when short sequencing reads (FASTQ, e.g. single-end HiSeq or ONT-derived
  reads) must be aligned to one known viral reference genome (FASTA) as the step preceding
  consensus sequence reconstruction or variant analysis.
when_to_use_negative:
- When reads are long or noisy native ONT/pacbio reads requiring a different minimap2
  preset (the -x sr preset is specifically for short reads)
- When the reference is an ultra-long genome (e.g. human chromosomes) where memory
  consumption for downstream consensus calling scales linearly with reference length
  and may be prohibitive
- When multiple distinct reference sequences must be handled jointly, since ViralConsensus-based
  consensus reconstruction requires a single reference sequence
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Maps short reads to the viral reference genome using -a -x sr to emit SAM
    alignments that feed consensus calling
- name: ViralConsensus
  role: Consumes the minimap2 SAM stream via standard input and calls the consensus
    genome on-the-fly
  repo: https://github.com/niemasd/ViralConsensus
- name: samtools
  role: Converts the unsorted mapper output stream to BAM (samtools view -b) when
    archiving an alignment file alongside piping
- name: htslib
  role: Core C library dependency that lets ViralConsensus read the alignment data
    produced by minimap2
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:0d0d445b2623211e213294c27663439f162ffdf3b97c6cd134b3766e33823e8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - htslib
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Short-read alignment with minimap2

## Summary

Map simulated or real short reads (e.g. Illumina HiSeq 2000) to a single viral reference genome with minimap2 in short-read mode, producing SAM/BAM alignments suitable for direct downstream consensus calling with ViralConsensus.

## When to use

Use when short sequencing reads (FASTQ, e.g. single-end HiSeq or ONT-derived reads) must be aligned to one known viral reference genome (FASTA) as the step preceding consensus sequence reconstruction or variant analysis.

## When NOT to use

- When reads are long or noisy native ONT/pacbio reads requiring a different minimap2 preset (the -x sr preset is specifically for short reads)
- When the reference is an ultra-long genome (e.g. human chromosomes) where memory consumption for downstream consensus calling scales linearly with reference length and may be prohibitive
- When multiple distinct reference sequences must be handled jointly, since ViralConsensus-based consensus reconstruction requires a single reference sequence

## Inputs

- Short-read FASTQ file(s), e.g. single-end ART-simulated HiSeq 2000 reads or NanoSim-H simulated ONT reads (optionally gzip-compressed)
- Single-sequence reference genome in FASTA format (e.g. SARS-CoV-2 lineage reference such as NC_045512.2, LC650844, OQ523614.1, OQ194009.1, or OQ346068.1)

## Outputs

- SAM alignment stream from minimap2 (-a output), optionally converted to BAM with samtools view
- Unsorted or sorted alignment file (BAM) usable by downstream consensus callers

## How to apply

Run minimap2 against a single reference sequence (e.g. a SARS-CoV-2 lineage genome such as NC_045512.2) using the short-read preset with single-threading: minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz. The -a flag emits SAM alignment output and -x sr selects parameters tuned for short reads. Because ViralConsensus can consume mapper output directly from standard input, pipe the unsorted SAM stream straight into viral_consensus to avoid unnecessary disk I/O, or interpose samtools view -b -@ 1 when a coordinate-archived BAM is needed. In benchmarking workflows (task_002), each simulated read set from ART (Illumina) or NanoSim-H (ONT) is mapped to its own source lineage reference genome so that consensus accuracy is measured against the correct ground-truth reference.

## Related tools

- **minimap2** (Maps short reads to the viral reference genome using -a -x sr to emit SAM alignments that feed consensus calling)
- **ViralConsensus** (Consumes the minimap2 SAM stream via standard input and calls the consensus genome on-the-fly) — https://github.com/niemasd/ViralConsensus
- **samtools** (Converts the unsorted mapper output stream to BAM (samtools view -b) when archiving an alignment file alongside piping)
- **htslib** (Core C library dependency that lets ViralConsensus read the alignment data produced by minimap2) — https://github.com/samtools/htslib

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- minimap2 command includes both -a (SAM output) and -x sr (short-read preset); omitting -a yields no alignable downstream input
- Alignment is produced against the same single reference FASTA that is later passed to viral_consensus via -r, so consensus coordinates match the reference
- The SAM/BAM stream is valid input for ViralConsensus (viral_consensus runs to completion and writes a FASTA consensus without errors from htslib)
- Piped invocation avoids intermediate SAM files on disk, matching the on-the-fly streaming design described in the paper
- In the benchmark setting, each read set maps only to its source lineage reference, and normalized Hamming error of the resulting consensus is computable via MAFFT alignment to that reference

## Limitations

- The -x sr preset is tuned for short reads; read types outside short-read sequencing would need a different minimap2 preset
- Downstream ViralConsensus memory consumption scales linearly with reference genome length, so alignments to ultra-long references (e.g. human chromosomes) may require prohibitively large memory
- minimap2 output is unsorted; tools needing coordinate-sorted BAM require an intermediate sorting/conversion step (e.g. samtools view -b)

## Evidence

- [methods] Short-read mapping command with sr preset piping into ViralConsensus: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [readme] Standard-input piping capability of ViralConsensus: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] samtools BAM conversion of mapper output: "samtools view -b -@ 1 > reads.bam"
- [methods] Avoiding disk I/O by piping directly: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
