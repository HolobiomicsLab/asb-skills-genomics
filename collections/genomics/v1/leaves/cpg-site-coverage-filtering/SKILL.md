---
name: cpg-site-coverage-filtering
description: Use when constructing controlled methylation mixture datasets from nanopore
  reads and you need to pick well-covered, single-CpG genomic sites before partitioning
  reads into methylated/unmethylated pools.
when_to_use_negative:
- The goal is per-site methylation frequency reporting on naturally methylated genomes
  rather than constructing controlled benchmarking mixtures from methylated/unmethylated
  controls
- Sites of interest contain multiple CpG dinucleotides within the analysis window,
  so per-site assignments would be confounded
- Coverage is below 50x, making per-site methylation frequency estimates unreliable
  for benchmarking
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3577
tools:
- name: Minimap2
  role: Aligns PASS nanopore reads from both control samples to the ecoli_k12_mg1655.fasta
    reference to enable per-site coverage computation
- name: samtools
  role: Sorts/indexes the aligned BAMs and computes per-site CpG read coverage used
    for the >=50x filtering threshold
- name: Snakemake
  role: Reproducible workflow orchestration for alignment, indexing, and coverage-filtering
    steps
  repo: https://github.com/comprna/METEORE
- name: METEORE
  role: Downstream consumer of the filtered site sets; mixture read assignments are
    written as per-read score TSVs (ID, Chr, Pos, Strand, Score) feeding methylation-calling
    tools and combination models
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:88db1de32a02dc2d838dddeb65b141112b032f69435f733f4efc05721b48fe77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - METEORE
  - Minimap2
  - Snakemake
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cpg-site-coverage-filtering

## Summary

Selects CpG sites suitable for controlled-methylation benchmarking by requiring a minimum per-site read coverage (>=50x, computed with samtools on Minimap2-aligned E. coli K12 nanopore BAMs) and retaining only sites with exactly one CpG dinucleotide within a 20 nt window, so that per-site methylation calls are unambiguous.

## When to use

Use when constructing controlled methylation mixture datasets from nanopore reads and you need to pick well-covered, single-CpG genomic sites before partitioning reads into methylated/unmethylated pools.

## When NOT to use

- The goal is per-site methylation frequency reporting on naturally methylated genomes rather than constructing controlled benchmarking mixtures from methylated/unmethylated controls
- Sites of interest contain multiple CpG dinucleotides within the analysis window, so per-site assignments would be confounded
- Coverage is below 50x, making per-site methylation frequency estimates unreliable for benchmarking

## Inputs

- PASS nanopore reads from PCR-amplified (unmethylated) E. coli K12 ER2925 sample
- PASS nanopore reads from M.SssI-treated (methylated) E. coli K12 ER2925 sample
- ecoli_k12_mg1655.fasta reference genome
- Minimap2-aligned, samtools-sorted/indexed BAM files for both samples

## Outputs

- Set of 100 single-CpG (20 nt window) sites at >=50x coverage for benchmarking dataset 1
- Set of 50 single-CpG (20 nt window) sites at >=50x coverage for benchmarking dataset 2
- Per-site CpG coverage table computed with samtools
- Read assignments to selected sites for methylated and unmethylated read pools

## How to apply

Align the PASS reads from the PCR-amplified (unmethylated) and M.SssI-treated (methylated) E. coli K12 ER2925 samples to the ecoli_k12_mg1655.fasta reference using Minimap2, then sort/index the BAMs with samtools. Compute per-site CpG read coverage with samtools and keep only CpG sites covered at >=50x. From these covered sites, retain only those containing exactly one CpG dinucleotide within a 20 nt window: 100 such sites for benchmarking dataset 1 and 50 for dataset 2. Rationale: enforcing a coverage floor ensures per-site methylation frequency estimates are stable across the 11 mixture sets (0%-100% methylated reads), and the single-CpG-in-window constraint prevents signal from neighbouring CpGs from confounding per-site comparisons of methylation-calling tools. Partition reads overlapping the selected sites into methylated and unmethylated pools for downstream mixture assembly in METEORE per-read TSV format (ID, Chr, Pos, Strand, Score).

## Related tools

- **Minimap2** (Aligns PASS nanopore reads from both control samples to the ecoli_k12_mg1655.fasta reference to enable per-site coverage computation)
- **samtools** (Sorts/indexes the aligned BAMs and computes per-site CpG read coverage used for the >=50x filtering threshold)
- **Snakemake** (Reproducible workflow orchestration for alignment, indexing, and coverage-filtering steps) — https://github.com/comprna/METEORE
- **METEORE** (Downstream consumer of the filtered site sets; mixture read assignments are written as per-read score TSVs (ID, Chr, Pos, Strand, Score) feeding methylation-calling tools and combination models) — https://github.com/comprna/METEORE

## Examples

```
minimap2 -ax ecoli_k12_mg1655.fasta reads.pass.fastq | samtools sort -o aln.bam && samtools index aln.bam && samtools depth -r <cpg_region> aln.bam
```

## Evaluation signals

- Selected site sets contain exactly 100 sites (dataset 1) and 50 sites (dataset 2), each with a single CpG dinucleotide in the 20 nt window
- Every retained site has read coverage >=50x as computed by samtools
- Reads overlapping selected sites partition cleanly into methylated (M.SssI-treated) and unmethylated (PCR-amplified) pools
- Each downstream dataset yields exactly 11 mixtures spanning 0%, 10%, ..., 100% methylated read proportions with ~2400 reads per mixture set
- Per-read score files conform to the METEORE TSV schema (ID, Chr, Pos, Strand, Score)

## Limitations

- The M.SssI treatment used to generate the CpG-methylated control only has ~95% efficiency, so 'methylated' pool reads are not fully methylated and per-site predictions may show low agreement with expected methylation frequency
- Thresholds (>=50x coverage, one CpG per 20 nt window) restrict the site universe and were chosen for benchmarking, not for general methylation analysis

## Evidence

- [methods] Compute per-site CpG read coverage with samtools and retain only CpG sites covered at >=50x.: [evidence span withheld — anchor status: not_found]
- [methods] single-CpG 20 nt-window site selection: [evidence span withheld — anchor status: not_found]
- [methods] Alignment rationale for coverage computation: [evidence span withheld — anchor status: not_found]
- [readme] Per-read score file format for downstream combination models: "Prepare the per-read score file from the methods you select"
