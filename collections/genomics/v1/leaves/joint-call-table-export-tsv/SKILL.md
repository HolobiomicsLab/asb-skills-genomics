---
name: joint-call-table-export-tsv
description: Use when you have .pytor files for multiple samples (e.g., a trio) and
  need to combine their CNV calls into joint regions, genotype contributing samples,
  and export a merged, annotated call table (TSV/XLS/VCF).
when_to_use_negative:
- Analyzing a single sample in isolation — use 'print calls' rather than 'print merged_calls',
  since merging requires multiple sample .pytor files passed to -root
- Somatic/tumor-normal CNA discovery — the article states a dedicated somatic CNA
  caller prototype exists (JointCaller example) and a more accurate cancer/mosaic
  approach is still under development
- Genotyping a small set of already-known regions across samples — use the -genotype
  command directly instead of building a full merged-call table
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CNVpytor
  role: 'Primary tool: -view mode loads multiple .pytor files, filters calls, clusters
    them into joint regions by >50% reciprocal overlap, annotates, and prints the
    merged table to TSV'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor read-depth CNV caller whose core engine (RD binning, mean-shift,
    partitioning, calling) CNVpytor inherits; calls from CNVnator and CNVpytor overlap
    by >99%
  repo: https://github.com/abyzovlab/CNVpytor
- name: Jupyter notebook
  role: CNVpytor can be used from Python in shared/cloud environments such as Jupyter
    notebook and Google Colab (CEPH trio example dataset) for scripted joint-call
    export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:2542d1f8b5b50e3096a5008d3c0847bfab6974368af26d9ce21c2fdee2b39403
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# joint-call-table-export-tsv

## Summary

Merge CNV calls from multiple samples into joint calls with CNVpytor's viewer mode (reciprocal overlap >50%), optionally annotate them, and export the merged call table to a tab-delimited TSV via set print_filename. This is CNVpytor's multi-sample joint-calling mechanism for trio/cohort CNV consolidation.

## When to use

Use when you have .pytor files for multiple samples (e.g., a trio) and need to combine their CNV calls into joint regions, genotype contributing samples, and export a merged, annotated call table (TSV/XLS/VCF).

## When NOT to use

- Analyzing a single sample in isolation — use 'print calls' rather than 'print merged_calls', since merging requires multiple sample .pytor files passed to -root
- Somatic/tumor-normal CNA discovery — the article states a dedicated somatic CNA caller prototype exists (JointCaller example) and a more accurate cancer/mosaic approach is still under development
- Genotyping a small set of already-known regions across samples — use the -genotype command directly instead of building a full merged-call table

## Inputs

- Two or more CNVpytor .pytor files (one per sample, e.g., CEPH trio samples) with RD-derived calls already computed via -rd/-his/-partition/-call
- Viewer-mode filter settings: Q0_range (e.g., 0 0.5), size_range (e.g., 100000 inf), optional p_range (e.g., 0 0.00001)
- Optional reference genome settings for Ensembl REST annotation

## Outputs

- Tab-delimited merged-joint-call table (output.tsv) with coordinates, contributing sample membership, and call statistics per joint region
- Optionally annotated merged calls (gene/feature annotations from Ensembl REST API overlap/region)
- Alternative export formats: output.xls spreadsheet or output.vcf via set print_filename

## How to apply

Launch CNVpytor in viewer mode passing all sample .pytor files to -root together with -view at a bin size such as 100000, which loads every sample into one comparison session. In interactive mode, apply call filters with 'set Q0_range 0 0.5', 'set size_range 100000 inf', and optionally 'set p_range 0 0.00001' to restrict calls before merging. Run 'print merged_calls': CNVpytor sorts calls by start coordinate and clusters calls from different samples whose regions have reciprocal overlap >50% into joint regions. To export, set 'set print_filename output.tsv' (or .xls/.vcf), optionally enable 'set annotate' to add Ensembl REST API overlap/region annotations, and re-run 'print merged_calls' so the table is written to file instead of stdout. Per-sample copy numbers in the table derive from genotyping logic where copy number is the mean RD in the region divided by mean autosomal RD scaled by 2 (available separately via -genotype 10000 100000).

## Related tools

- **CNVpytor** (Primary tool: -view mode loads multiple .pytor files, filters calls, clusters them into joint regions by >50% reciprocal overlap, annotates, and prints the merged table to TSV) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor read-depth CNV caller whose core engine (RD binning, mean-shift, partitioning, calling) CNVpytor inherits; calls from CNVnator and CNVpytor overlap by >99%) — https://github.com/abyzovlab/CNVpytor
- **Jupyter notebook** (CNVpytor can be used from Python in shared/cloud environments such as Jupyter notebook and Google Colab (CEPH trio example dataset) for scripted joint-call export)

## Examples

```
cnvpytor -root file1.pytor file2.pytor file3.pytor -view 100000; set Q0_range 0 0.5; set size_range 100000 inf; set print_filename output.tsv; set annotate; print merged_calls
```

## Evaluation signals

- output.tsv exists and is tab-delimited with coordinate columns, sample-membership fields, and per-region call statistics
- Every reported merged region satisfies the clustering criterion: calls from different samples with reciprocal overlap >50% are grouped into one joint region
- All exported calls pass the applied filters (Q0 within 0–0.5, size ≥ 100000 bp, and p ≤ 0.00001 if p_range was set)
- When 'set annotate' was used, merged-call rows include Ensembl annotation fields in addition to coordinates and sample membership
- Row count of merged_calls output is ≤ the total number of per-sample calls, since clustering collapses reciprocally overlapping calls across samples

## Limitations

- Annotation via Ensembl REST API is an optional step that requires a web connection
- The 5 viewer-mode filter parameters (CNV size, e-val1, q0, pN, and dG) control what enters the merged table; overly permissive settings inflate joint regions
- Genotype copy numbers are RD-based (mean RD / mean autosomal RD scaled by 2); regions failing SNP/BAF support may be less reliable
- Merged calls reflect the bin size chosen at -view launch (e.g., 100000), so resolution is bounded by that binning

## Evidence

- [discussion] Merging calls over multiple samples with reciprocal overlap >50%: "select calls from other samples with reciprocal overlap >50%"
- [discussion] Viewer-mode call filtering parameters: "There are 5 parameters in viewer mode used for filtering calls: CNV size, e-val1, q0, pN, and dG."
- [discussion] Genotyping copy-number definition: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
- [methods] Filter and export commands for annotated output: "set print_filename output.tsv
set annotate
print calls"
- [readme] README multi-sample merging invocation: "> cnvpytor -root file1.pytor file2.pytor ... -view 100000 
print merged_calls"
