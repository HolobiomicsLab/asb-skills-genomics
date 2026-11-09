---
name: assembly-contiguity-assessment
description: Use when comparing a draft assembly (e.g. celegans_flye.fa) against one
  or more scaffolder outputs (e.g. *ntLink.scaffolds.fa, per-round ntLink_rounds intermediates)
  to quantify N50/NG50 gains and misassembly trade-offs.
when_to_use_negative:
- Input is raw long reads (e.g. SRR10028109 ONT FASTQ) rather than assembled contig/scaffold
  FASTAs — run the scaffolder first.
- The goal is to perform scaffolding or gap-filling itself (Basic Protocols 1–3) rather
  than to evaluate the contiguity of existing assemblies.
- The claim of interest is sequence-level base accuracy rather than contiguity/misassembly
  metrics — use a polishing/consensus evaluation instead.
edam_operation: http://edamontology.org/operation_0286
edam_topics: []
tools:
- name: abyss-fac (ABySS)
  role: Computes total length, N50, and sequence counts for the draft and each scaffolded
    assembly version
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: Misassembly-aware evaluation in --split-scaffold mode, yielding NG50, genome
    fraction, and misassembly counts against the reference
  repo: https://github.com/ablab/quast
- name: ntLink
  role: Produces the scaffolded assemblies (single-round, gap-filling, or 3-round
    runs) that are being assessed
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:c5b9bf3fdc222c8243b32b52d3c0b347cb2d114948d4963e443e73d3d855cca5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/abyss
  tools:
  - QUAST
  - abyss-fac (ABySS)
  - minimap2
  - ntLink
  edam_operation: http://edamontology.org/operation_0286
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# assembly-contiguity-assessment

## Summary

Quantifies contiguity improvements of draft vs. scaffolded genome assemblies using abyss-fac (ABySS v2.3.0+) for N50/total-length statistics and QUAST (with --split-scaffold) for NG50, genome fraction, and misassembly-aware metrics, as demonstrated in the ntLink C. elegans Flye protocols.

## When to use

Use when comparing a draft assembly (e.g. celegans_flye.fa) against one or more scaffolder outputs (e.g. *ntLink.scaffolds.fa, per-round ntLink_rounds intermediates) to quantify N50/NG50 gains and misassembly trade-offs.

## When NOT to use

- Input is raw long reads (e.g. SRR10028109 ONT FASTQ) rather than assembled contig/scaffold FASTAs — run the scaffolder first.
- The goal is to perform scaffolding or gap-filling itself (Basic Protocols 1–3) rather than to evaluate the contiguity of existing assemblies.
- The claim of interest is sequence-level base accuracy rather than contiguity/misassembly metrics — use a polishing/consensus evaluation instead.

## Inputs

- Draft assembly FASTA (e.g. celegans_flye.fa from Flye)
- Scaffolded assembly FASTA(s) with the *ntLink.scaffolds.fa suffix and/or per-round intermediates with the target.k<w>.w<w>.z<z> prefix scheme
- Reference genome for QUAST (e.g. C. elegans GCA_000002985.3)

## Outputs

- abyss-fac statistics table (total length, N50, sequence counts) per assembly
- QUAST report with NG50, genome fraction, and misassembly counts (from --split-scaffold mode)
- Per-round contiguity comparison table across scaffolding/gap-filling iterations

## How to apply

Compute assembly statistics for the input draft and every scaffolded output with abyss-fac (from ABySS v2.3.0+), recording N50 and total scaffold length. Then run QUAST in --split-scaffold mode on the same FASTAs to break scaffolds at Ns and obtain misassembly-aware contiguity metrics (NG50, genome fraction, misassembly counts) aligned against the reference (e.g. C. elegans GCA_000002985.3). For iterative runs, tabulate per-round metrics from the intermediate assemblies named with the target.k<w>.w<w>.z<z> prefix scheme (e.g. k=24, w=250, rounds=3) to quantify gains contributed by scaffolding versus gap-filling. Correct application is judged by the expected signature: NG50/N50 increases after scaffolding, the number of sequences decreases, and misassembly counts do not rise substantially.

## Related tools

- **abyss-fac (ABySS)** (Computes total length, N50, and sequence counts for the draft and each scaffolded assembly version) — https://github.com/BirolLab/abyss
- **QUAST** (Misassembly-aware evaluation in --split-scaffold mode, yielding NG50, genome fraction, and misassembly counts against the reference) — https://github.com/ablab/quast
- **ntLink** (Produces the scaffolded assemblies (single-round, gap-filling, or 3-round runs) that are being assessed) — https://github.com/bcgsc/ntLink

## Examples

```
abyss-fac celegans_flye.fa celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa; quast.py --split-scaffold -r GCA_000002985.3 celegans_flye.fa celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa
```

## Evaluation signals

- N50 and total scaffold length increase after scaffolding relative to the Flye draft, per abyss-fac.
- The number of sequences decreases after scaffolding (contigs merged into scaffolds).
- QUAST NG50 and genome fraction reproduce the Table 3-reported gains when aligning ntLink scaffolds to GCA_000002985.3.
- Misassembly counts do not increase substantially after scaffolding or gap-filling rounds.
- Per-round metrics show the expected pattern across 3 rounds of ntLink_rounds with gap-filling (k=24, w=250), with each round's intermediate assessable via abyss-fac.

## Limitations

- abyss-fac reports raw scaffold contiguity without breaking at Ns; QUAST --split-scaffold is needed for misassembly-aware (gap-adjusted) contiguity, so both views should be tabulated.
- QUAST NG50 and genome fraction require a reference genome; without one, only N50-style statistics from abyss-fac are available.
- Contiguity gains must be interpreted alongside misassembly counts, since joins suggested by long-read mapping evidence can be incorrect.

## Evidence

- [abstract] we present three basic protocols demonstrating how to use each of these new features to yield highly contiguous genome assemblies, while still maintaining ntLink's proven computational efficiency: "we present three basic protocols demonstrating how to use each of these new features to yield highly contiguous genome assemblies, while still maintaining ntLink's proven computational efficiency"
- [abstract] ntLink utilizes long-read sequencing data to improve upon draft genome assemblies built from any sequencing technologies, including the same long reads: [evidence span withheld — anchor status: not_found]
- [readme] ABySS is a *de novo* sequence assembler intended for short paired-end reads and genomes of all sizes.: [evidence span withheld — anchor status: not_found]
