---
name: dorado-methylation-calling
description: Use when benchmarking nanopore CpG methylation callers on R10.4 human
  or plant datasets, or when an independent ONT-official baseline (Dorado) methylation
  call set is needed to compare against a deep-learning caller such as DeepBAM.
when_to_use_negative:
- 'Data is from older pore versions (e.g., pre-R10.4): the article notes only a small
  number of tools, including Dorado and DeepMod2, are compatible with the newer R10.4
  chemistry'
- The goal is single-molecule methylation state accuracy (AUROC/F1) rather than frequency-level
  benchmarking — DeepBAM was shown to outperform Dorado on R10.4 single-molecule calls
- Input is already a methylation frequency table; this skill operates on raw/down-sampled
  nanopore BAM reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: ONT's official methylation caller; produces the comparator CpG modification
    calls on the down-sampled datasets
- name: samtools
  role: Down-samples the nanopore BAM datasets to 10x and 30x coverage with samtools
    view -s before methylation calling
- name: Oxford Nanopore Technology (ONT)
  role: Sequencing platform generating the R10.4 nanopore datasets used for calling
- name: DeepBAM
  role: Deep-learning CpG methylation caller run in parallel as the method being benchmarked
    against Dorado
  repo: github.com/xiaochuanle/DeepBAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:0c3043c3f1e2bb542a9f3ee7144df8db1ee73645a0a5f87fd582b222e0039441
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  - Oxford Nanopore Technology (ONT)
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dorado-methylation-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Baseline CpG methylation calling on Oxford Nanopore R10.4 data using ONT's official Dorado tool, used as the comparator against DeepBAM for single-molecule modification calls and genome-wide methylation frequency profiles benchmarked against BS-seq.

## When to use

Use when benchmarking nanopore CpG methylation callers on R10.4 human or plant datasets, or when an independent ONT-official baseline (Dorado) methylation call set is needed to compare against a deep-learning caller such as DeepBAM.

## When NOT to use

- Data is from older pore versions (e.g., pre-R10.4): the article notes only a small number of tools, including Dorado and DeepMod2, are compatible with the newer R10.4 chemistry
- The goal is single-molecule methylation state accuracy (AUROC/F1) rather than frequency-level benchmarking — DeepBAM was shown to outperform Dorado on R10.4 single-molecule calls
- Input is already a methylation frequency table; this skill operates on raw/down-sampled nanopore BAM reads

## Inputs

- Down-sampled nanopore BAM files (R10.4 ONT reads) at 10x and 30x coverage
- Reference genome FASTA for the corresponding human cell line or plant species
- Matched BS-seq methylation frequency tracks for validation

## Outputs

- Per-site CpG modification calls from Dorado
- Genome-wide CpG methylation frequency profiles for Dorado
- Correlation table (bs_seq_correlation_table.csv) of Pearson coefficients versus BS-seq for Dorado and DeepBAM at each coverage

## How to apply

Run Dorado methylation calling on down-sampled nanopore BAM datasets (e.g., samtools view -s to 10x and 30x coverage) for each dataset under evaluation, alongside the competing caller (DeepBAM's extract_and_call_mods with motif_type=CG, kmer_size=51). Aggregate Dorado's per-site CpG modification calls into genome-wide methylation frequency profiles and compute Pearson correlation coefficients against matched BS-seq methylation frequency tracks. The rationale is that the article found Dorado's methylation calling model 'remains poorly tested' and 'performed significantly different across datasets', so running it identically on every down-sampled dataset (human cell lines including HG001 and plant species including O. sativa) establishes a fair, dataset-stratified baseline. In the reference evaluation, Dorado was outperformed by DeepBAM in all instances at both 10x (BS-seq correlations 0.9032–0.9847) and 30x (0.9212–0.9872) coverage.

## Related tools

- **Dorado** (ONT's official methylation caller; produces the comparator CpG modification calls on the down-sampled datasets)
- **samtools** (Down-samples the nanopore BAM datasets to 10x and 30x coverage with samtools view -s before methylation calling)
- **Oxford Nanopore Technology (ONT)** (Sequencing platform generating the R10.4 nanopore datasets used for calling)
- **DeepBAM** (Deep-learning CpG methylation caller run in parallel as the method being benchmarked against Dorado) — github.com/xiaochuanle/DeepBAM

## Examples

```
samtools view -s 0.1 samples.bam > sample_10x.bam && dorado basecaller sup sample_10x.bam | dorado aligner ref.fa > sample_10x_meth.bam
```

## Evaluation signals

- bs_seq_correlation_table.csv exists with rows for both coverages (10x, 30x) and both callers (Dorado, DeepBAM)
- Dorado's Pearson correlations with BS-seq are computed for all five datasets (human cell lines including HG001 and plant species including O. sativa)
- Expected pattern from the reference evaluation: DeepBAM correlations exceed Dorado in every instance at both depths (DeepBAM 10x range 0.9032–0.9847)
- Down-sampled BAMs verify at the intended coverages (10x and 30x) after samtools view -s
- Dorado calls use the same CpG motif and down-sampled inputs as the DeepBAM extract_and_call_mods runs to ensure a controlled comparison

## Limitations

- Dorado's methylation calling model 'remains poorly tested' and its performance 'differ[s] significantly across heterogeneous human and plant datasets'
- DeepBAM outperformed Dorado in all instances, so Dorado alone is not the most accurate caller for R10.4 CpG methylation
- DeepBAM's own performance is acknowledged as needing 'ongoing optimization', and quantitative speed/memory comparisons were deferred to supplementary material

## Evidence

- [methods] Dorado as baseline comparator in benchmark workflow: [evidence span withheld — anchor status: not_found]
