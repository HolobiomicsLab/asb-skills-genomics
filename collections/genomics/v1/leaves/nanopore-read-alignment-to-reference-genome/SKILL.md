---
name: nanopore-read-alignment-to-reference-genome
description: Use when raw nanopore PASS reads (e.g. PCR-amplified or M.SssI-treated
  E. coli K12 samples in fast5/fastq) must be mapped to a reference such as ecoli_k12_mg1655.fasta
  before methylation-calling or coverage filtering.
when_to_use_negative:
- Input reads are already aligned — an indexed BAM exists, so re-alignment adds no
  information before methylation calling.
- The sequencing platform is short-read (Illumina WGBS); Minimap2 nanopore presets
  and this long-read pipeline are inappropriate for such data.
- No reference genome is available or the target genome differs substantially from
  the provided ecoli_k12_mg1655.fasta without a valid substitute.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Minimap2
  role: Aligns nanopore PASS reads to the ecoli_k12_mg1655.fasta reference genome
    to produce SAM/BAM alignments
- name: samtools
  role: Sorts and indexes the Minimap2 BAM output and computes per-site CpG read coverage
    for the >=50x filter
- name: Snakemake
  role: Orchestrates the alignment and downstream methylation-calling rules reproducibly
    in the METEORE pipelines
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:eda99703005ae2d42526b68453acc905337c122e1bad0da2029390070ae74a76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - Guppy
  - METEORE
  - Minimap2
  - Snakemake
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Nanopore read alignment to reference genome

## Summary

Aligns Oxford Nanopore long reads to a reference genome with Minimap2 and processes the resulting BAM with samtools, forming the foundational coordinate anchor for downstream methylation calling in the METEORE benchmarking pipelines.

## When to use

Use when raw nanopore PASS reads (e.g. PCR-amplified or M.SssI-treated E. coli K12 samples in fast5/fastq) must be mapped to a reference such as ecoli_k12_mg1655.fasta before methylation-calling or coverage filtering.

## When NOT to use

- Input reads are already aligned — an indexed BAM exists, so re-alignment adds no information before methylation calling.
- The sequencing platform is short-read (Illumina WGBS); Minimap2 nanopore presets and this long-read pipeline are inappropriate for such data.
- No reference genome is available or the target genome differs substantially from the provided ecoli_k12_mg1655.fasta without a valid substitute.

## Inputs

- PASS nanopore reads (single-read fast5 / fastq) from methylated (M.SssI-treated) and unmethylated (PCR-amplified) E. coli K12 ER2925 samples
- Reference genome in FASTA format (e.g. data/ecoli_k12_mg1655.fasta)

## Outputs

- Sorted, coordinate-indexed BAM files (one per sample) via samtools sort/index
- Per-site CpG read coverage counts computed with samtools for >=50x coverage filtering

## How to apply

Align the PASS reads from each sample (e.g. the PCR-amplified unmethylated and M.SssI-treated methylated E. coli K12 ER2925 control sets) to the ecoli_k12_mg1655.fasta reference using Minimap2 in nanopore-appropriate mode, then sort and index the resulting BAM files with samtools so downstream tools can query them. After indexing, compute per-site read coverage with samtools to support threshold filtering — in the benchmarking-mixture construction this meant retaining only CpG sites covered at >=50x. Placement of the reference .fasta in a `data` folder and referencing it inside each tool Snakefile is how the METEORE pipelines parameterize the alignment target. Sorted, indexed BAMs are the required input format for methylation callers like Nanopolish, DeepSignal, and Tombo in the workflow.

## Related tools

- **Minimap2** (Aligns nanopore PASS reads to the ecoli_k12_mg1655.fasta reference genome to produce SAM/BAM alignments)
- **samtools** (Sorts and indexes the Minimap2 BAM output and computes per-site CpG read coverage for the >=50x filter)
- **Snakemake** (Orchestrates the alignment and downstream methylation-calling rules reproducibly in the METEORE pipelines) — https://github.com/comprna/METEORE

## Examples

```
minimap2 -ax map-ont data/ecoli_k12_mg1655.fasta reads.fq | samtools sort -o aligned.bam && samtools index aligned.bam
```

## Evaluation signals

- Each sample yields a sorted, coordinate-indexed BAM that downstream Snakemake rules (Nanopolish, DeepSignal, Tombo, Guppy Snakefiles) can successfully consume.
- Per-site CpG coverage computed with samtools identifies sites at >=50x coverage as expected before the single-CpG 20 nt-window site selection step.
- Aligned reads partition cleanly into the methylated (M.SssI-treated) and unmethylated (PCR-amplified) read pools used to build the 11 mixture benchmarking datasets at 0–100% methylated proportions with ~2400 reads each.
- The reference genome path matches the FASTA re-defined inside each tool Snakefile (e.g. ecoli_k12_mg1655.fasta) so all tools align against the same reference.

## Limitations

- The METEORE example data are E. coli K12 positive-control reads; applying to other organisms requires replacing the reference FASTA in the `data` folder and re-defining it in every Snakefile.
- Some downstream methylation tools (e.g. Megalodon) require GPUs to be time efficient, which constrains the practical scale of the alignment-plus-calling pipeline.
- Guppy methylation fast5 calls require external conversion scripts (e.g. gcf52ref) to reference-anchored output, and the author notes this process 'is not very efficient and the output is not very useful' in that prototype.

## Evidence

- [other] Align the PASS reads from the PCR-amplified (unmethylated) and M.SssI-treated (methylated) E. coli K12 ER2925 samples to the ecoli_k12_mg1655.fasta reference using Minimap2: [evidence span withheld — anchor status: not_found]
- [methods] We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta`: "We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta`"
- [readme] You should place the reference genome file in *.fasta* format in a folder named `data`: "You should place the reference genome file in *.fasta* format in a folder named `data`"
- [other] Compute per-site CpG read coverage with samtools and retain only CpG sites covered at >=50x.: [evidence span withheld — anchor status: not_found]
