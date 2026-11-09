---
name: cnv-call-filtering-by-q0-size-pvalue
description: Use when CNV calls already stored in .pytor files must be narrowed by
  quality (q0), size (e.g. ≥100 kb), and p-value (e.g. ≤1e-5) thresholds before printing,
  annotating, or multi-sample merging.
when_to_use_negative:
- Input is a raw BAM/CRAM without a .pytor file — the -rd, -his, -partition, and -call
  steps must run first before any call filtering can be applied
- The variants to filter are SNPs and small indels in a VCF (handled by the -snp/-baf
  workflow), not CNV calls in .pytor storage
- The goal is somatic CNA discovery in cancer or mosaic samples, which relies on the
  prototype somatic joint caller rather than viewer-mode call filtering
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: CNVpytor
  role: Provides viewer mode (-view) with set Q0_range / set size_range / set p_range
    commands to filter called CNVs and print the result to a file
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor RD-based CNV caller whose calls CNVpytor reimplements and extends
    with viewer-mode filtering; used as a comparison baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:a5680b241abe0b0d735ef1d9fd4af5e8ce554b51b47f8d9587f621f3d236bf24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Jupyter notebook
  - Python
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cnv-call-filtering-by-q0-size-pvalue

## Summary

Filters CNV calls stored in CNVpytor .pytor files using viewer-mode thresholds on Q0 (RD quality), CNV size, and p-value before printing, annotating, or merging calls across samples.

## When to use

Use when CNV calls already stored in .pytor files must be narrowed by quality (q0), size (e.g. ≥100 kb), and p-value (e.g. ≤1e-5) thresholds before printing, annotating, or multi-sample merging.

## When NOT to use

- Input is a raw BAM/CRAM without a .pytor file — the -rd, -his, -partition, and -call steps must run first before any call filtering can be applied
- The variants to filter are SNPs and small indels in a VCF (handled by the -snp/-baf workflow), not CNV calls in .pytor storage
- The goal is somatic CNA discovery in cancer or mosaic samples, which relies on the prototype somatic joint caller rather than viewer-mode call filtering

## Inputs

- CNVpytor .pytor file (HDF5) containing called CNV regions from a sample
- Bin size for viewer mode (e.g. 100000), matching the bin size used during -call
- Filter parameter values: Q0_range (e.g. 0 0.5), size_range (e.g. 100000 inf), p_range (e.g. 0 0.00001)

## Outputs

- Filtered CNV call table written via set print_filename (output.tsv, output.xls, or output.vcf)
- Optionally annotated calls (Ensembl REST API overlap/region annotations) when set annotate is active
- Filtered merged_calls table when viewer mode is launched with multiple .pytor files

## How to apply

Enter CNVpytor viewer mode with the bin size used for calling (e.g. `cnvpytor -root file.pytor -view 100000`), then set the filter criteria interactively: `set Q0_range 0 0.5` removes calls with high Q0 (poor RD-based quality), `set size_range 100000 inf` keeps only CNVs ≥100 kb, and `set p_range 0 0.00001` applies a stricter p-value cutoff. Viewer mode exposes five filter parameters — CNV size, e-val1, q0, pN, and dG — and each subsequent `print calls` reflects the currently active thresholds. Direct the filtered output with `set print_filename output.tsv` (or output.xls / output.vcf) and optionally enable `set annotate` to decorate calls via the Ensembl REST API overlap/region resource. The same filtering applies to `print merged_calls` when combining calls from multiple samples, since filtering is a prerequisite step for joint-calling.

## Related tools

- **CNVpytor** (Provides viewer mode (-view) with set Q0_range / set size_range / set p_range commands to filter called CNVs and print the result to a file) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor RD-based CNV caller whose calls CNVpytor reimplements and extends with viewer-mode filtering; used as a comparison baseline)

## Examples

```
cnvpytor -root file.pytor -view 100000; set Q0_range 0 0.5; set size_range 100000 inf; set p_range 0 0.00001; set print_filename output.tsv; print calls
```

## Evaluation signals

- Every retained call in the printed table satisfies the active thresholds: q0 ≤ 0.5, size ≥ 100000 bp, and p ≤ 0.00001 when p_range is set
- Comparing `print calls` before and after the `set` commands yields a reduced call set consistent with the thresholds applied
- The file specified by set print_filename (e.g. output.tsv) exists and is a tab-delimited table of filtered calls
- With set annotate enabled, printed calls carry Ensembl annotation fields, confirming the optional annotation step ran with web access

## Limitations

- Viewer mode exposes only five filter parameters — CNV size, e-val1, q0, pN, and dG — so filtering is limited to these criteria
- Annotation via the Ensembl REST API is an optional step that requires a web connection
- A more accurate approach for discovering cancer and mosaic CNAs is still under development, so viewer-mode filtering is not a substitute for a dedicated somatic joint caller

## Evidence

- [readme] set Q0_range 0 0.5
set size_range 100000 inf
set print_filename output.tsv
set annotate
print calls: "set Q0_range 0 0.5
set size_range 100000 inf
set print_filename output.tsv
set annotate
print calls"
- [discussion] There are 5 parameters in viewer mode used for filtering calls: CNV size, e-val1, q0, pN, and dG.: "There are 5 parameters in viewer mode used for filtering calls: CNV size, e-val1, q0, pN, and dG."
- [abstract] CNV calls can be filtered using several criteria, annotated, and merged over multiple samples: "CNV calls can be filtered using several criteria, annotated, and merged over multiple samples"
