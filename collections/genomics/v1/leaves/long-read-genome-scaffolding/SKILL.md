---
name: long-read-genome-scaffolding
description: Use when a draft assembly (FASTA contigs) needs improved contiguity and
  matching long reads (e.g. ONT FASTQ such as SRR10028109) are available to provide
  mapping-supported contig joins.
when_to_use_negative:
- The draft assembly is already highly contiguous and long-read assemblers (e.g. Flye)
  have fully exhausted the long-range evidence — ntLink targets leftover join opportunities
- Only short paired-end reads are available and no long-read library exists for the
  sample, since ntLink scaffolding relies on long-read minimizer mappings
- The goal is misassembly correction rather than scaffolding — use ntLink mappings
  with Tigmint-long (Alternate Protocol 2) instead of the scaffold mode
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Primary minimizer-based long-read scaffolder; computes sketches, maps reads,
    finds joins, applies overlap detection and gap-filling
  repo: https://github.com/bcgsc/ntLink
- name: ABySS
  role: abyss-scaffold (v2.3.0+) traverses the ntLink scaffold graph; abyss-fac computes
    assembly contiguity statistics
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: Evaluates scaffolded assemblies (NG50, genome fraction, misassemblies) against
    a reference, with --split-scaffold for gap-filled scaffolds
- name: minimap2
  role: Long-read mapper used alongside ntLink in the protocol workflows
- name: btllib
  role: Required dependency library (1.7.1 or higher) for ntLink
  repo: https://github.com/BirolLab/btllib
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:576a345572a87cdc9b51d74c98ae48fd7e6a6e7aae2be763788ec2bcd0162930
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - QUAST
  - btllib
  - minimap2
  - ntLink
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# long-read-genome-scaffolding

## Summary

Scaffold a draft genome assembly (e.g. a Flye contig FASTA) using long-read sequencing data with ntLink, a lightweight minimizer-based scaffolder that infers contig ordering/orientation from minimizer mappings rather than read alignments, optionally with overlap detection, gap-filling, or multiple in-code scaffolding rounds.

## When to use

Use when a draft assembly (FASTA contigs) needs improved contiguity and matching long reads (e.g. ONT FASTQ such as SRR10028109) are available to provide mapping-supported contig joins.

## When NOT to use

- The draft assembly is already highly contiguous and long-read assemblers (e.g. Flye) have fully exhausted the long-range evidence — ntLink targets leftover join opportunities
- Only short paired-end reads are available and no long-read library exists for the sample, since ntLink scaffolding relies on long-read minimizer mappings
- The goal is misassembly correction rather than scaffolding — use ntLink mappings with Tigmint-long (Alternate Protocol 2) instead of the scaffold mode

## Inputs

- Draft genome assembly FASTA (e.g. celegans_flye.fa from Flye)
- Long-read sequencing data in FASTQ (gzipped OK, e.g. SRR10028109.fq.gz ONT reads)
- Optional reference genome (e.g. C. elegans GCA_000002985.3) for QUAST-based NG50/genome-fraction assessment
- ABySS v2.3.0+ and btllib 1.7.1+ installed for abyss-scaffold/abyss-fac support

## Outputs

- Scaffolded assembly FASTA with the *ntLink.scaffolds.fa suffix (e.g. celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- Per-round intermediate scaffold FASTAs for iterative runs (named with the target.k<w>.w<w>.z<z> prefix scheme)
- abyss-fac statistics (total length, N50, sequence counts) for draft and scaffolded assemblies
- QUAST metrics (NG50, genome fraction, misassembly counts) versus a reference

## How to apply

Place the target assembly (e.g. celegans_flye.fa) and long reads (e.g. SRR10028109.fq.gz) in the working directory, soft-linking files if necessary. ntLink first computes ordered minimizer sketches of the target and reads, maps reads to contigs via minimizers, and finds contig pairs whose joins are supported by long-read mapping evidence, then outputs a scaffold graph traversed by abyss-scaffold (ABySS v2.3.0+). Basic Protocol 1 runs a single scaffolding pass with overlap=True (defaults z=1000, n=1, a=1) to trim overlapping joined sequences, using parameters like k=32 w=100 t=5. For gap-filling, run Basic Protocol 2 mode where gaps are filled with raw read sequence; for iterative gains, use the ntLink_rounds Makefile run_rounds_gaps target (e.g. k=24 w=250 rounds=3), which lifts mappings between rounds to reduce compute. Evaluate contiguity gains with abyss-fac (total length, N50, sequence counts) and QUAST (NG50, genome fraction, misassemblies; use --split-scaffold for gap-filled scaffolds) against a reference such as C. elegans GCA_000002985.3, expecting increased N50/NG50 and decreased sequence counts without a large rise in misassemblies.

## Related tools

- **ntLink** (Primary minimizer-based long-read scaffolder; computes sketches, maps reads, finds joins, applies overlap detection and gap-filling) — https://github.com/bcgsc/ntLink
- **ABySS** (abyss-scaffold (v2.3.0+) traverses the ntLink scaffold graph; abyss-fac computes assembly contiguity statistics) — https://github.com/BirolLab/abyss
- **QUAST** (Evaluates scaffolded assemblies (NG50, genome fraction, misassemblies) against a reference, with --split-scaffold for gap-filled scaffolds)
- **minimap2** (Long-read mapper used alongside ntLink in the protocol workflows)
- **btllib** (Required dependency library (1.7.1 or higher) for ntLink) — https://github.com/BirolLab/btllib

## Examples

```
ntLink scaffold target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5 overlap=True
```

## Evaluation signals

- Output FASTA exists with the *ntLink.scaffolds.fa suffix naming scheme (target.k<w>.w<w>.z<z>.ntLink.scaffolds.fa)
- abyss-fac shows NG50/N50 increase and number of sequences decrease relative to the input draft (Table 3-style contiguity gains)
- QUAST shows genome fraction maintained or improved with no large increase in misassemblies versus the draft
- For gap-filled rounds, QUAST --split-scaffold per-round metrics show monotonic contiguity gains across the 3 rounds
- Per-round intermediate FASTAs following the target.k<w>.w<w>.z<z> prefix scheme are produced for iterative runs

## Limitations

- Gap-filling inserts raw read sequence into N gaps, so QUAST should be run in --split-scaffold mode to avoid overstating contiguity
- Defaults (z=1000, n=1, a=1) and k/w parameters must be appropriate for read length and assembly fragmentation; different protocols use different k/w (k=32 w=100 for overlap mode, k=24 w=250 for rounds mode)
- All assembly and read files must be present in the working directory (soft links if necessary) or the Makefile-based rounds mode will not resolve inputs

## Evidence

- [abstract] ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"
- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] Compute ordered minimizer sketches of the input target assembly and long reads: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [methods] Output a scaffold graph, where the nodes are oriented contigs and the edges are joins suggested by the long read data: "4. Output a scaffold graph, where the nodes are oriented contigs and the edges are joins suggested by the long read data"
- [readme] [ABySS v2.3.0+](https://github.com/BirolLab/abyss): "[btllib](https://github.com/BirolLab/btllib) 1.7.1 or higher"
