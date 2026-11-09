---
name: illumina-read-quality-trimming
description: Use when Illumina paired-end FASTQ reads (e.g., 300 bp MiSeq PE at >250-fold
  coverage) are the input to a short-read assembler such as SPAdes whose contigs will
  be scaffolded by npScarf, and adapters/low-quality bases must be removed first.
when_to_use_negative:
- Input reads are nanopore long reads — npScarf consumes these via BWA-MEM streaming
  alignment, not short-read trimming
- The Illumina data are already trimmed — the npScarf project distributes pre-trimmed
  files such as Kp2146_paired_1.fastq.gz, and re-trimming trimmed data is unnecessary
- The assembly pipeline does not start from a short-read draft (e.g., long-read-only
  assembly with Celera Assembler or npGraph)
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: trimmomatic
  role: Removes adapters and low-quality bases from Illumina MiSeq paired-end reads
    prior to SPAdes assembly
- name: SPAdes
  role: Assembles the trimmed paired-end reads into draft contigs that npScarf scaffolds
    in real-time
- name: npScarf
  role: Consumes the trimmed-read-derived draft assembly (jsa.np.npscarf -seq) for
    real-time nanopore scaffolding
  repo: https://github.com/mdcao/npScarf
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:fa322101c855cef145359b119bedf7e23077b390e92f4f124c4aff80bfd36ed4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - SPAdes
  - npReader
  - npScarf
  - trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# illumina-read-quality-trimming

## Summary

Trim Illumina MiSeq paired-end reads to strip adapters and low-quality bases with trimmomatic V0.32 before short-read assembly, so that the draft contigs fed into npScarf real-time scaffolding are free of adapter-induced misassemblies.

## When to use

Use when Illumina paired-end FASTQ reads (e.g., 300 bp MiSeq PE at >250-fold coverage) are the input to a short-read assembler such as SPAdes whose contigs will be scaffolded by npScarf, and adapters/low-quality bases must be removed first.

## When NOT to use

- Input reads are nanopore long reads — npScarf consumes these via BWA-MEM streaming alignment, not short-read trimming
- The Illumina data are already trimmed — the npScarf project distributes pre-trimmed files such as Kp2146_paired_1.fastq.gz, and re-trimming trimmed data is unnecessary
- The assembly pipeline does not start from a short-read draft (e.g., long-read-only assembly with Celera Assembler or npGraph)

## Inputs

- Raw Illumina MiSeq paired-end FASTQ files (e.g., ENA ERR654977, 300 bp paired-end, >250-fold coverage)
- Adapter/quality-trimming configuration for trimmomatic V0.32

## Outputs

- Trimmed paired FASTQ files with adapters and low-quality bases removed (e.g., Kp2146_paired_1.fastq.gz / Kp2146_paired_2.fastq.gz)
- Downstream SPAdes draft assembly (contigs.fasta) sorted by reverse length (Kp2146_spades.fasta)

## How to apply

Take the raw Illumina MiSeq paired-end FASTQ files (e.g., ENA ERR654977 for E. coli K12 MG1655, or the Kpn2146 tutorial data) and trim them with trimmomatic V0.32 to remove all adapters and low-quality bases; the npScarf README states this is essential. Feed the trimmed paired FASTQ into SPAdes (>= 3.5, multi-threaded, --careful optional) to produce contigs.fasta, then sort contigs in reverse length order with 'jsa.seq.sort -r -n' before building the bwa index. The resulting largest contigs (>20 Kb) are what npScarf uses to estimate contig multiplicity (threshold θ = 1.5) and to reach a single circular scaffold under 30-fold nanopore coverage, so trimming quality directly gates the real-time termination statistic.

## Related tools

- **trimmomatic** (Removes adapters and low-quality bases from Illumina MiSeq paired-end reads prior to SPAdes assembly)
- **SPAdes** (Assembles the trimmed paired-end reads into draft contigs that npScarf scaffolds in real-time)
- **npScarf** (Consumes the trimmed-read-derived draft assembly (jsa.np.npscarf -seq) for real-time nanopore scaffolding) — https://github.com/mdcao/npScarf

## Evaluation signals

- Trimmed output FASTQ retains read-pair synchrony between the two mate files and contains no residual adapter sequence
- SPAdes on the trimmed reads yields contigs including a stable set of the up to 20 largest contigs longer than 20 Kb, which npScarf requires for its termination statistic
- jsa.seq.sort -r -n output places the longest contigs first, confirming the draft is ready for bwa index and npScarf -seq input
- Final QUAST report on npScarf output (e.g., EcK12S.fin.fasta) shows one complete circular contig matching the E. coli K12 MG1655 reference, indicating upstream trimming did not degrade the assembly
- npScarf reaches a single circular scaffold with under 30-fold nanopore coverage, showing the short-read draft was clean enough for low-coverage scaffolding

## Limitations

- SPAdes --careful mode gives only marginal error reduction; the npScarf README notes 'the improvement is not so significant' so it can be excluded to save runtime
- Trimming applies only to the Illumina short-read arm; the overall npScarf pipeline still depends on a nanopore base-caller and minute-level (not millisecond-level) real-time resolution
- The npScarf software is no longer maintained; npGraph is the successor, so this trimming step feeds a legacy pipeline

## Evidence

- [discussion] Read data from Illumina sequencing were trimmed with trimmomatic V0.32 (ref. 41): "Read data from Illumina sequencing were trimmed with trimmomatic V0.32 (ref. 41)"
- [readme] Illumina reads must be trimmed before SPAdes/npScarf: "It is essential that the reads are trimmed to remove all adaptors. Low quality bases should also be removed."
- [discussion] Sequencing depth and read length of the Illumina input: "sequenced on the MiSeq instrument (Illumina) with 300 bp paired end sequencing, to a coverage of over 250-fold."
- [abstract] Why trimmed short-read drafts matter for npScarf: "Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress."
