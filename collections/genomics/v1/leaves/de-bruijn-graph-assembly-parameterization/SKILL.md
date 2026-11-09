---
name: de-bruijn-graph-assembly-parameterization
description: Use when configuring de novo transcriptome assembly of Illumina RNA-Seq
  reads and you need to compute (rather than hand-pick) k-mer sizes for iterative
  de Bruijn graph construction from the read length.
when_to_use_negative:
- The input is not transcriptomic data — SPAdes' genome/metagenome pipelines and their
  fixed k series (21, 33, 55, 77) apply instead
- Input reads are high-error-rate long reads (Oxford Nanopore/PacBio), which cannot
  be used directly as complete transcripts and require hybrid or long-read pipelines
- A fixed k schedule is already validated for the dataset and the goal is comparability
  with prior SPAdes-based runs rather than automatic selection
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: rnaSPAdes
  role: Transcriptome assembler that computes the k-mer schedule automatically from
    read length
  repo: github.com/ablab/spades
- name: SPAdes
  role: Base assembler whose iterative de Bruijn graph framework and default k series
    (21, 33, 55, 77) rnaSPAdes modifies
  repo: github.com/ablab/spades
- name: rnaQUAST
  role: Quality evaluation of de novo transcriptome assemblies produced with the chosen
    k schedule
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:706e8664d7c70b32a255c1a9e0a4550c42d195dfc3209e36366042d3dbbb2e93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPAdes
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# de-bruijn-graph-assembly-parameterization

## Summary

Selects multi-k de Bruijn graph iteration schedules for RNA-Seq assembly, specifically reproducing rnaSPAdes' automatic two-value k-mer schedule derived from input read length, in contrast to SPAdes' fixed developer-chosen k series (e.g., 21, 33, 55, 77).

## When to use

Use when configuring de novo transcriptome assembly of Illumina RNA-Seq reads and you need to compute (rather than hand-pick) k-mer sizes for iterative de Bruijn graph construction from the read length.

## When NOT to use

- The input is not transcriptomic data — SPAdes' genome/metagenome pipelines and their fixed k series (21, 33, 55, 77) apply instead
- Input reads are high-error-rate long reads (Oxford Nanopore/PacBio), which cannot be used directly as complete transcripts and require hybrid or long-read pipelines
- A fixed k schedule is already validated for the dataset and the goal is comparability with prior SPAdes-based runs rather than automatic selection

## Inputs

- Paired-end Illumina RNA-Seq reads (FASTQ or BAM for error correction)
- SRA metadata containing per-dataset read lengths (e.g., SRR5133163, SRR1957703, SRX648736, SRR1560107, SRR1588569, SRR5344669)

## Outputs

- Two-value k-mer schedule (lower k ≈ read_length/3, min 29; upper k = largest odd ≤ read_length/2 - 1)
- De novo transcriptome assembly (contigs/transcripts) assembled with the computed k schedule

## How to apply

First, extract the read length of the input dataset (e.g., from SRA metadata for accessions such as Human SRR5133163, Human large SRR1957703/SRR1957706, Mouse SRX648736, Worm SRR1560107, Corn SRR1588569, Arabidopsis SRR5344669/SRR5344670). Compute the upper k as the largest odd number that does not exceed read_length/2 - 1, i.e., nearly half the read length, so that graph edges retain sufficient overlap between reads. Compute the lower k as approximately read_length/3 with a minimum of 29: values below 29 dramatically increase erroneous contigs from transcripts sharing the same k-mer. rnaSPAdes uses this automatic two-value schedule instead of SPAdes' longer iterative series (21, 33, 55, 77); validate by checking the computed pair reproduces the assembler's documented default behavior for that read length.

## Related tools

- **rnaSPAdes** (Transcriptome assembler that computes the k-mer schedule automatically from read length) — github.com/ablab/spades
- **SPAdes** (Base assembler whose iterative de Bruijn graph framework and default k series (21, 33, 55, 77) rnaSPAdes modifies) — github.com/ablab/spades
- **rnaQUAST** (Quality evaluation of de novo transcriptome assemblies produced with the chosen k schedule)

## Examples

```
rnaspades.py --pe1-1 SRR5133163_1.fastq.gz --pe1-2 SRR5133163_2.fastq.gz -o rnaspades_human_out
```

## Evaluation signals

- Upper k equals the largest odd number ≤ read_length/2 - 1 for every dataset's read length
- Lower k is approximately read_length/3 and never below 29
- Computed schedule reproduces rnaSPAdes' documented default k-values for that read length (no unmatched dataset)
- Assembly with small first k shows no excess of erroneous contigs from transcripts sharing the same k-mer
- Resulting assembly quality checked with rnaQUAST metrics (e.g., fraction of 95%-assembled genes/isoforms) comparable to rnaSPAdes defaults

## Limitations

- Using small k (e.g., k=21) during the first iteration can assemble extremely low-covered genes where read overlaps are short, but small k-mer sizes in transcriptome assembly dramatically increase erroneous contigs from transcripts sharing the same k-mer
- The schedule is derived from a single read-length assumption; mixed-length or non-standard reads fall outside the stated formula
- The approach was designed for RNA-Seq from a single organism; metatranscriptome application (repetitive homologous gene patterns, extreme mRNA quantity differences) is unaddressed

## Evidence

- [methods] iterative k selection rationale: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] small k trade-off: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [readme] scope of SPAdes package: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
- [intro] rnaQUAST role: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
