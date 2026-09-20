---
name: giab-variant-annotation-of-reference-genome
description: 'Use when building a truth-anchored simulation benchmark: a reference
  (e.g. chr22/hg38) must be annotated with GiAB NA12878 variants so that SNV/indel
  recall and precision of downstream variant callers can be scored.'
when_to_use_negative:
- You need de novo variant discovery from real patient data — this skill benchmarks
  simulators against a truth set, not clinical calling.
- Your reference already carries the truth variants (e.g. the modified chr22 itself)
  — aligning to it would hide simulated variants and invalidate recall scoring.
- Sequencing platforms other than nanopore (e.g. Illumina or PacBio) — the seq2squiggle/DeepSimulator/squigulator
  simulation and pore profiles used here are nanopore-specific.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: seq2squiggle
  role: Simulates nanopore signals (BLOW5/POD5) from the NA12878-annotated chr22 reference,
    e.g. 250,000 R10.4.1 reads and 50,000 R9.4.1 reads
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Comparison simulator used to generate signals from the same modified chr22
    reference
- name: DeepSimulator
  role: Comparison simulator run in two distinct configurations on the same reference
- name: minimap2
  role: Aligns basecalled reads to hg38 so simulated variants can be recovered as
    candidate calls
- name: Clair3
  role: Calls SNVs and indels from each simulator's alignment
- name: RTGtools
  role: Scores each Clair3 callset against the NA12878 truth variants, producing TP/FP/recall/precision
- name: mamba
  role: Package manager used to install tool environments for the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:ca45ca5ade534932ea4ea63c4625774b8054c277c9656da559a0014663639cdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - DeepSimulator
  - RTGtools
  - mamba
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# giab-variant-annotation-of-reference-genome

## Summary

Constructs a variant-carrying reference by applying Genome in a Bottle NA12878 variants to hg38 chr22, then benchmarks nanopore read simulators by basecalling, aligning, and variant-calling their output against the GiAB truth set with Clair3 and RTGtools.

## When to use

Use when building a truth-anchored simulation benchmark: a reference (e.g. chr22/hg38) must be annotated with GiAB NA12878 variants so that SNV/indel recall and precision of downstream variant callers can be scored.

## When NOT to use

- You need de novo variant discovery from real patient data — this skill benchmarks simulators against a truth set, not clinical calling.
- Your reference already carries the truth variants (e.g. the modified chr22 itself) — aligning to it would hide simulated variants and invalidate recall scoring.
- Sequencing platforms other than nanopore (e.g. Illumina or PacBio) — the seq2squiggle/DeepSimulator/squigulator simulation and pore profiles used here are nanopore-specific.

## Inputs

- hg38 chr22 reference sequence (FASTA)
- Genome in a Bottle NA12878 variant truth set for chr22 (VCF)
- Simulated nanopore signal datasets in SLOW5/BLOW5 or POD5 format (R9.4.1 and R10.4.1 chemistries)
- Basecalled read sequences (FASTQ) from simulated signals

## Outputs

- Modified variant-carrying chr22 reference (FASTA with NA12878 variants applied)
- Per-simulator alignment files (BAM/PAF from minimap2)
- Per-simulator variant callsets (VCF from Clair3)
- variant_calling_benchmark.tsv with true positives, false positives, recall, and precision per simulator

## How to apply

First build a modified chr22 reference by applying Genome in a Bottle NA12878 variants onto hg38; this makes true-positive variant expectations known by construction. From this modified reference, simulate raw nanopore signal (e.g. 250,000 R10.4.1 reads and 50,000 R9.4.1 reads with seq2squiggle; comparable simulations with squigulator and two DeepSimulator configurations) and basecall each signal dataset into read sequences. Align the basecalled reads back to the unmodified hg38 with minimap2 so simulated variants appear as genuine candidate variants rather than alignment artifacts. Call variants from each alignment with Clair3, then score every callset against the NA12878 truth variants with RTGtools, compiling true positives, false positives, recall, and precision per simulator (e.g. into variant_calling_benchmark.tsv). The rationale is that higher true-positive SNV counts and better recall/precision indicate a simulator whose signals produce more realistic basecalled reads.

## Related tools

- **seq2squiggle** (Simulates nanopore signals (BLOW5/POD5) from the NA12878-annotated chr22 reference, e.g. 250,000 R10.4.1 reads and 50,000 R9.4.1 reads) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Comparison simulator used to generate signals from the same modified chr22 reference)
- **DeepSimulator** (Comparison simulator run in two distinct configurations on the same reference)
- **minimap2** (Aligns basecalled reads to hg38 so simulated variants can be recovered as candidate calls)
- **Clair3** (Calls SNVs and indels from each simulator's alignment)
- **RTGtools** (Scores each Clair3 callset against the NA12878 truth variants, producing TP/FP/recall/precision)
- **mamba** (Package manager used to install tool environments for the pipeline)

## Examples

```
seq2squiggle predict chr22_NA12878.fasta -o sim_r104.blow5 -n 250000
```

## Evaluation signals

- Each simulator's callset scores against the identical NA12878 truth variants, so TP/FP/recall/precision are directly comparable across simulators.
- Alignments must target unmodified hg38, not the variant-annotated chr22, or simulated variants will be masked and recall artificially deflated.
- Read counts per simulator should match the design (250,000 R10.4.1; 50,000 R9.4.1) to keep coverage comparable.
- Expected result pattern: seq2squiggle yields more true-positive SNVs and higher recall and precision than squigulator and DeepSimulator (Fig. 4).
- The compiled variant_calling_benchmark.tsv should contain one row with TP, FP, recall, and precision for each simulator configuration.

## Limitations

- Benchmark is restricted to chr22 and NA12878 truth variants, so conclusions may not transfer to other genomes or samples.
- Results depend on basecaller behavior; simulators are compared only through the full basecall-align-call chain, not on raw-signal fidelity alone.
- seq2squiggle's development testing was 'Not implemented yet' per the README, and optimal simulation performance requires a GPU for high read counts.

## Evidence

- [methods] Workflow: apply GiAB variants to build truth-anchored reference: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: truth-based scoring of variant callsets: [evidence span withheld — anchor status: not_found]
- [abstract] Key comparative finding that the benchmark is designed to reveal: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
- [readme] README: role of seq2squiggle in the simulation step: "seq2squiggle is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
