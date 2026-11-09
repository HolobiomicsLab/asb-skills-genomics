---
name: cpg-methylation-frequency-computation
description: Use when you have per-site CpG methylation calls from nanopore reads
  (e.g., a DeepBAM or Dorado-modified BAM at a defined coverage such as 10x-30x) plus
  matched BS-seq frequency tracks, and need genome-wide methylation frequency profiles
  and BS-seq correlation coefficients.
when_to_use_negative:
- You only need single-molecule read-level methylation state classification (AUROC/F1
  evaluation), not aggregated genome-wide frequency comparison
- No matched BS-seq methylation frequency tracks exist for the dataset, so correlation
  cannot be computed
- Input data is not CpG-motif nanopore R10.4 data (e.g., other methylation motifs
  or PacBio kinetics data)
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Calls CpG modifications (extract_and_call_mods mode, motif_type=CG) to produce
    per-site methylation rates used for frequency aggregation
  repo: https://github.com/xiaochuanle/DeepBAM
- name: Dorado
  role: ONT baseline methylation caller run on the same down-sampled datasets to obtain
    comparable CpG modification calls
- name: samtools
  role: Down-samples nanopore BAM datasets to target coverage (e.g., samtools view
    -s for 10x and 30x)
- name: Oxford Nanopore Technology (ONT)
  role: Sequencing platform supplying the R10.4 nanopore data used in the frequency
    computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:1d17c6c1f97ce07c77bc02c551c037d12a7933c92d6d12f9da448d3263c672c9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  - Oxford Nanopore Technology (ONT)
  - samtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cpg-methylation-frequency-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute genome-wide CpG methylation frequency profiles from single-molecule nanopore methylation calls (e.g., DeepBAM or Dorado on R10.4 data) and quantify their fidelity against bisulfite sequencing (BS-seq) via Pearson correlation. This skill turns per-site modification calls into aggregate methylation frequencies suitable for benchmarking across human and plant datasets.

## When to use

Use when you have per-site CpG methylation calls from nanopore reads (e.g., a DeepBAM or Dorado-modified BAM at a defined coverage such as 10x-30x) plus matched BS-seq frequency tracks, and need genome-wide methylation frequency profiles and BS-seq correlation coefficients.

## When NOT to use

- You only need single-molecule read-level methylation state classification (AUROC/F1 evaluation), not aggregated genome-wide frequency comparison
- No matched BS-seq methylation frequency tracks exist for the dataset, so correlation cannot be computed
- Input data is not CpG-motif nanopore R10.4 data (e.g., other methylation motifs or PacBio kinetics data)

## Inputs

- Nanopore BAM file with R10.4 reads (human cell lines such as HG001, or plant species such as O. sativa)
- Reference genome for the corresponding species
- Matched BS-seq methylation frequency tracks (per-CpG profiles)
- Target down-sampling coverages (e.g., 10x and 30x)

## Outputs

- Genome-wide CpG methylation frequency profiles per caller and per down-sampled dataset
- Pearson correlation coefficients between BS-seq and each caller's methylation frequencies
- A correlation summary table (e.g., bs_seq_correlation_table.csv with rows for both coverages and both callers)

## How to apply

First down-sample each nanopore BAM to the target coverage (e.g., 10x and 30x) with `samtools view -s`. Run methylation calling on each down-sampled dataset — e.g., DeepBAM's extract_and_call_mods mode with the CG motif (motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10) against the reference genome, and Dorado on the same inputs for comparison. Aggregate per-site methylation rates into genome-wide CpG methylation frequency profiles for each caller. Load matched BS-seq methylation frequency tracks and compute Pearson correlation coefficients between BS-seq and each caller's frequencies. Expect DeepBAM correlations above 0.95 on most datasets (10x range 0.9032 HG001 to 0.9847 O. sativa), exceeding Dorado in every comparison.

## Related tools

- **DeepBAM** (Calls CpG modifications (extract_and_call_mods mode, motif_type=CG) to produce per-site methylation rates used for frequency aggregation) — https://github.com/xiaochuanle/DeepBAM
- **Dorado** (ONT baseline methylation caller run on the same down-sampled datasets to obtain comparable CpG modification calls)
- **samtools** (Down-samples nanopore BAM datasets to target coverage (e.g., samtools view -s for 10x and 30x))
- **Oxford Nanopore Technology (ONT)** (Sequencing platform supplying the R10.4 nanopore data used in the frequency computation)

## Examples

```
samtools view -s 0.1 samples.bam > 10x.bam; deepbam extract_and_call_mods --motif_type CG --kmer_size 51 --batch_size 1024 --num_workers 10 -i 10x.bam -r reference.fa
```

## Evaluation signals

- DeepBAM-derived methylation frequencies achieve Pearson correlation >0.95 with BS-seq on four of five datasets at 10x depth
- DeepBAM correlations exceed Dorado correlations at every depth and dataset tested
- At 10x coverage correlations fall in the range 0.9032 (HG001) to 0.9847 (O. sativa); at 30x, 0.9212 (HG001) to 0.9872 (O. sativa)
- The output correlation table exists in CSV format with rows for both coverages (10x, 30x) and both callers (DeepBAM, Dorado)
- Down-sampled BAMs verify at the intended coverage (10x or 30x) before methylation calling is run

## Limitations

- Dorado methylation calling performance differs significantly across heterogeneous human and plant datasets, so caller comparisons must be made per-dataset rather than averaged naively
- DeepBAM correlations are lowest for HG001 (~0.90 at 10x), so results on some human cell lines may not exceed the 0.95 threshold
- Ongoing optimization of DeepBAM is acknowledged as necessary, and only 9-kmer modules are fully updated so far, implying incomplete current performance for other k-mer sizes

## Evidence
