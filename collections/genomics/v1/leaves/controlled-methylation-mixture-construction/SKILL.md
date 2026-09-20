---
name: controlled-methylation-mixture-construction
description: Use when building ground-truth benchmarking mixtures of methylated and
  unmethylated nanopore reads to evaluate per-read or per-site methylation callers,
  or when preparing METEORE-format score files with known expected methylation frequencies.
when_to_use_negative:
- You need per-read methylation predictions on natural samples with unknown methylation
  status rather than benchmarking against a known ground truth
- Coverage at candidate CpG sites is below the 50x threshold, so methylated/unmethylated
  read pools cannot be reliably partitioned at unambiguous single-CpG loci
- Evaluating tools on WGBS or Cas9-targeted (nCATS) data instead of controlled mixtures
  of individual nanopore reads
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3295
tools:
- name: Minimap2
  role: Aligns methylated and unmethylated control reads to the ecoli_k12_mg1655 reference
    to identify reads overlapping selected CpG sites
- name: samtools
  role: Sorts/indexes BAMs and computes per-site CpG read coverage to filter sites
    at >=50x
- name: Snakemake
  role: Wraps the pipeline steps for reproducible mixture construction
  repo: https://github.com/comprna/METEORE
- name: METEORE
  role: Consumes the per-read score files in its TSV format (ID, Chr, Pos, Strand,
    Score) for downstream methylation-calling tools and combination models
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:0fc74759f9e32d3d91118fb4518e1898206acadb60120e40e75c0ceeecd8ed89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - METEORE
  - Minimap2
  - Snakemake
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# controlled-methylation-mixture-construction

## Summary

Constructs controlled methylation mixture benchmarking datasets from nanopore reads of E. coli methylated (M.SssI-treated) and unmethylated (PCR-amplified) controls, enabling ground-truth evaluation of CpG methylation detection tools at known methylated-read proportions.

## When to use

Use when building ground-truth benchmarking mixtures of methylated and unmethylated nanopore reads to evaluate per-read or per-site methylation callers, or when preparing METEORE-format score files with known expected methylation frequencies.

## When NOT to use

- You need per-read methylation predictions on natural samples with unknown methylation status rather than benchmarking against a known ground truth
- Coverage at candidate CpG sites is below the 50x threshold, so methylated/unmethylated read pools cannot be reliably partitioned at unambiguous single-CpG loci
- Evaluating tools on WGBS or Cas9-targeted (nCATS) data instead of controlled mixtures of individual nanopore reads

## Inputs

- PASS fast5/fastq reads from M.SssI-treated (CpG-methylated) E. coli K12 ER2925 control sample
- PASS fast5/fastq reads from PCR-amplified (unmethylated) E. coli K12 ER2925 control sample
- E. coli reference genome ecoli_k12_mg1655.fasta

## Outputs

- 11 controlled methylation mixtures per dataset (0%–100% methylated in 10% steps, ~2400 reads per mixture) for dataset 1 (100 single-CpG sites) and dataset 2 (50 sites)
- Per-read mixture assignments in METEORE per-read TSV format (ID, Chr, Pos, Strand, Score) with known expected methylation frequencies

## How to apply

Align PASS reads from the M.SssI-treated (methylated) and PCR-amplified (unmethylated) E. coli K12 ER2925 samples to ecoli_k12_mg1655.fasta using Minimap2, then sort/index with samtools. Compute per-CpG coverage and retain only CpG sites covered at >=50x; from these, select sites containing exactly one CpG dinucleotide within a 20 nt window (100 sites for dataset 1, 50 sites for dataset 2) so that per-read methylation status at the site is unambiguous. Partition reads overlapping the selected sites into a methylated pool (M.SssI-treated) and an unmethylated pool (PCR-amplified), then randomly sample reads from both pools to build 11 mixtures per dataset at 0%, 10%, ..., 100% methylated proportions with ~2400 reads each. Write per-dataset, per-mixture score assignments in the METEORE per-read TSV format (ID, Chr, Pos, Strand, Score) so downstream methylation-calling tools and combination models can be evaluated against known expected frequencies.

## Related tools

- **Minimap2** (Aligns methylated and unmethylated control reads to the ecoli_k12_mg1655 reference to identify reads overlapping selected CpG sites)
- **samtools** (Sorts/indexes BAMs and computes per-site CpG read coverage to filter sites at >=50x)
- **Snakemake** (Wraps the pipeline steps for reproducible mixture construction) — https://github.com/comprna/METEORE
- **METEORE** (Consumes the per-read score files in its TSV format (ID, Chr, Pos, Strand, Score) for downstream methylation-calling tools and combination models) — https://github.com/comprna/METEORE

## Examples

```
minimap2 -ax map-ont ecoli_k12_mg1655.fasta methylated_pass.fq | samtools sort -o methylated.bam; samtools index methylated.bam
```

## Evaluation signals

- Each dataset yields exactly 11 mixtures with methylated read proportions of 0%, 10%, 20%, ..., 100%
- Each mixture contains ~2400 reads
- Dataset 1 selected site set contains 100 single-CpG sites within 20 nt windows and dataset 2 contains 50, all at >=50x coverage
- Mixture read assignments written as per-dataset, per-mixture METEORE per-read TSV files (ID, Chr, Pos, Strand, Score) that parse correctly for downstream combination models
- Methylated reads trace exclusively to the M.SssI-treated pool and unmethylated reads to the PCR-amplified pool

## Limitations

- The M.SssI methylated control has only ~95% methylation efficiency, so mixtures nominally labeled 100% methylated may not be fully methylated, reducing agreement with expected methylation frequency
- Only CpG sites with a single CpG dinucleotide in a 20 nt window and >=50x coverage are eligible, restricting which loci can be benchmarked
- Benchmarking based on controlled mixtures assesses read-level frequency recovery; per-site predictions from individual tools showed high dispersion and did not generally agree with expected methylation frequency

## Evidence

- [abstract] controlled mixtures of methylated and unmethylated reads: "a systematic benchmarking of tools for the detection of CpG methylation from Nanopore sequencing using individual reads, control mixtures of methylated and unmethylated reads"
- [readme] single-read score files in METEORE format: "Prepare the per-read score file from the methods you select"
- [discussion] high dispersion of per-site predictions: "the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency"
- [discussion] M.SssI control ~95% efficiency: "A possible reason for this low agreement is that the M.Sssl treatment used to generate the CpG-methylated control only has 95% efficiency"
- [methods] minimap2/samtools alignment setup for mixture construction: "Download the scripts that convert CpG methylation from fast5s to reference anchored calls
git clone https://github.com/kpalin/gcf52ref.git"
