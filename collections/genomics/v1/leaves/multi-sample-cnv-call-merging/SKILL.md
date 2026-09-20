---
name: multi-sample-cnv-call-merging
description: Use when you have CNV calls from several related samples (e.g., trio
  or cohort .pytor files) and need to produce a unified joint call set with sample
  membership and per-sample genotypes rather than per-sample call lists.
when_to_use_negative:
- You need single-sample CNV calling from raw BAMs — this skill operates on already-called
  .pytor files in view mode, not on the -rd/-his/-partition/-call pipeline itself
- You need somatic CNA detection between tumor-normal pairs — the article notes the
  somatic CNA caller is only a functional prototype, distinct from multi-sample merged_calls
- Your samples' calls do not co-occur at reciprocal overlap >50%, in which case calls
  will simply not cluster into joint regions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: 'Primary tool: viewer mode merges calls across multiple .pytor files with
    reciprocal overlap >50% and prints joint call tables'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor whose RD-based calling engine (binning, mean-shift, partitioning,
    calling) underlies the merged calls
- name: Python
  role: Language in which CNVpytor is implemented; enables scripted use of the merging
    workflow
  repo: https://github.com/abyzovlab/CNVpytor
- name: Jupyter notebook
  role: Interactive environment for running CNVpytor (e.g., CEPH trio Colab example)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:1f48c4b92096f7921eb08042f05c0ab89a2f8eeb56fb1be3353a71418fd8d266
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Jupyter notebook
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-sample-cnv-call-merging

## Summary

Merges CNV calls from multiple samples (e.g., a CEPH trio of .pytor files) into joint calls by clustering regions with reciprocal overlap >50% in CNVpytor viewer mode, optionally followed by per-sample genotyping of the merged regions.

## When to use

Use when you have CNV calls from several related samples (e.g., trio or cohort .pytor files) and need to produce a unified joint call set with sample membership and per-sample genotypes rather than per-sample call lists.

## When NOT to use

- You need single-sample CNV calling from raw BAMs — this skill operates on already-called .pytor files in view mode, not on the -rd/-his/-partition/-call pipeline itself
- You need somatic CNA detection between tumor-normal pairs — the article notes the somatic CNA caller is only a functional prototype, distinct from multi-sample merged_calls
- Your samples' calls do not co-occur at reciprocal overlap >50%, in which case calls will simply not cluster into joint regions

## Inputs

- Multiple CNVpytor .pytor files (HDF5 storage), one per sample, e.g. from a CEPH/platinum-genomes trio
- Viewer-mode call filter settings (Q0_range, size_range, optionally p_range)
- Optional: target genomic regions for -genotype (e.g., 12:11396601-11436500)

## Outputs

- Merged joint call table (TSV/XLS/VCF) with coordinates, contributing sample membership, and per-sample genotype copy-number values
- Optional annotated merged calls via Ensembl REST API overlap/region resource
- Optional plots of all merged calls (PNG) via 'set print' and 'set output_filename prefix.png'

## How to apply

Launch CNVpytor in viewer mode on all samples at once, e.g. 'cnvpytor -root file1.pytor file2.pytor file3.pytor -view 100000', so calls from all samples are loaded into one view. Apply call filters in viewer mode — 'set Q0_range 0 0.5' and 'set size_range 100000 inf', optionally 'set p_range 0 0.00001' for stricter p-value filtering — to restrict the call pool before merging. Run 'print merged_calls': CNVpytor sorts calls by start coordinate and clusters calls from different samples that share reciprocal overlap >50% into joint regions. Genotype each merged region per sample with the -genotype command at bin sizes 10000 and 100000, which reports copy number as mean RD within the region divided by mean autosomal RD scaled by 2. Route the table to a file via 'set print_filename output.tsv' (with optional 'set annotate' for Ensembl REST API annotation) and print the merged joint call table containing coordinates, contributing samples, and per-sample genotype values.

## Related tools

- **CNVpytor** (Primary tool: viewer mode merges calls across multiple .pytor files with reciprocal overlap >50% and prints joint call tables) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor whose RD-based calling engine (binning, mean-shift, partitioning, calling) underlies the merged calls)
- **Python** (Language in which CNVpytor is implemented; enables scripted use of the merging workflow) — https://github.com/abyzovlab/CNVpytor
- **Jupyter notebook** (Interactive environment for running CNVpytor (e.g., CEPH trio Colab example))

## Examples

```
cnvpytor -root file1.pytor file2.pytor file3.pytor -view 100000 -c 'set Q0_range 0 0.5; set size_range 100000 inf; set print_filename output.tsv; print merged_calls'
```

## Evaluation signals

- output.tsv exists, is tab-delimited, and contains coordinate, sample-membership, and per-sample genotype fields per merged region
- Every reported merged region satisfies the >50% reciprocal-overlap clustering criterion across samples
- Genotype copy numbers fall in the expected range (e.g., ~2.0 for neutral regions, as in the README example '12:11396601-11436500  1.933261  1.937531')
- Filters are confirmed in effect via the viewer 'show' parameters (Q0_range 0 0.5, size_range 100000 inf, optional p_range 0 0.00001)
- Merged calls sorted by start coordinate cluster calls from all supplied samples, not just one

## Limitations

- CNVpytor is significantly faster than CNVnator for parsing alignments (2–20x), but multi-sample merging operates on pre-called .pytor files, not raw alignments
- Annotation via Ensembl REST API requires an active web connection and is optional
- More accurate approaches for cancer and mosaic CNAs are still under development; the somatic CNA caller is only a prototype
- Genotyping reports copy number as mean RD divided by mean autosomal RD scaled by 2, so quality depends on RD normalization and GC correction

## Evidence

- [abstract] CNV calls can be filtered using several criteria, annotated, and merged over multiple samples: "CNV calls can be filtered using several criteria, annotated, and merged over multiple samples"
- [readme] Merging calls from multiple samples (README): "> cnvpytor -root file1.pytor file2.pytor ... -view 100000 
print merged_calls"
- [discussion] Reciprocal overlap threshold: "select calls from other samples with reciprocal overlap >50%"
- [discussion] Genotyping definition: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
