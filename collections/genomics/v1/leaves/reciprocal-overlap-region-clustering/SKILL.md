---
name: reciprocal-overlap-region-clustering
description: Use when several samples have been individually CNV-called (e.g., .pytor
  files from a trio) and the goal is to produce joint multi-sample CNV calls by clustering
  calls with reciprocal overlap >50% and genotyping each merged region per sample.
when_to_use_negative:
- Only a single sample is being analyzed and no cross-sample merging is needed — use
  plain 'print calls' in viewer mode instead
- The task is somatic tumor-normal CNA calling — the article notes CNVpytor's somatic
  CNA caller is only a prototype and a more accurate approach for cancer/mosaic CNAs
  was still under development
- Calls cannot be overlapped because they come from incompatible genome builds or
  coordinate systems
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: 'Performs the multi-sample merging: viewer mode with -root over multiple .pytor
    files, ''print merged_calls'' clustering by reciprocal overlap >50%, and -genotype
    of merged regions'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor read-depth CNV caller whose core engine (RD binning, mean-shift,
    partitioning, calling) CNVpytor inherits; used to define the per-sample calls
    being merged
- name: Python
  role: Implementation language of CNVpytor; usable via Jupyter notebook/Google Colab
    for scripted merging workflows
- name: JBrowse / igv.js
  role: Visualization of CNVpytor .pytor files and merged calls
  repo: https://github.com/igvteam/igv.js/wiki/cnvpytor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:52f7659cd812e363be1730aa002aff72a837ab97d7031c5db841b077a34637cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - JBrowse / igv.js
  - Jupyter notebook
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reciprocal-overlap-region-clustering

## Summary

Joint-calling mechanism in CNVpytor that merges CNV calls from multiple samples (e.g., the CEPH trio) into joint regions by clustering calls with reciprocal overlap >50%, followed by per-sample genotyping of the merged regions.

## When to use

Use when several samples have been individually CNV-called (e.g., .pytor files from a trio) and the goal is to produce joint multi-sample CNV calls by clustering calls with reciprocal overlap >50% and genotyping each merged region per sample.

## When NOT to use

- Only a single sample is being analyzed and no cross-sample merging is needed — use plain 'print calls' in viewer mode instead
- The task is somatic tumor-normal CNA calling — the article notes CNVpytor's somatic CNA caller is only a prototype and a more accurate approach for cancer/mosaic CNAs was still under development
- Calls cannot be overlapped because they come from incompatible genome builds or coordinate systems

## Inputs

- One .pytor file per sample (e.g., three files for the CEPH trio) with RD-based CNV calls already computed via -rd/-his/-partition/-call
- Viewer-mode bin size (e.g., 100000)
- Optional call-filter parameters: Q0_range, size_range, p_range

## Outputs

- Table (output.tsv/xls/vcf) of merged joint-call regions sorted by start coordinate, with contributing sample membership and per-sample genotype copy-number values
- Optional annotated merged calls (via Ensembl REST API overlap/region resource)
- Optional PNG plots of all merged calls (set print with set output_filename prefix.png)

## How to apply

Launch CNVpytor viewer mode passing all sample .pytor files to -root with -view at a chosen bin size (e.g., 100000). Apply call filters in viewer mode such as 'set Q0_range 0 0.5', 'set size_range 100000 inf', and optionally 'set p_range 0 0.00001'. Run 'print merged_calls' so CNVpytor sorts calls by start coordinate and clusters calls from different samples with reciprocal overlap >50% into joint regions. Genotype each merged region per sample using the -genotype command at bin sizes 10000 and 100000; the reported copy number is the mean RD within the region divided by mean autosomal RD scaled by 2. Direct output with 'set print_filename output.tsv' and print the merged, annotated call table containing coordinates, contributing samples, and per-sample genotype values.

## Related tools

- **CNVpytor** (Performs the multi-sample merging: viewer mode with -root over multiple .pytor files, 'print merged_calls' clustering by reciprocal overlap >50%, and -genotype of merged regions) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor read-depth CNV caller whose core engine (RD binning, mean-shift, partitioning, calling) CNVpytor inherits; used to define the per-sample calls being merged)
- **Python** (Implementation language of CNVpytor; usable via Jupyter notebook/Google Colab for scripted merging workflows)
- **JBrowse / igv.js** (Visualization of CNVpytor .pytor files and merged calls) — https://github.com/igvteam/igv.js/wiki/cnvpytor

## Examples

```
cnvpytor -root file1.pytor file2.pytor file3.pytor -view 100000
```

## Evaluation signals

- Every reported merged region satisfies the >50% reciprocal-overlap clustering criterion across contributing samples
- output.tsv exists and is tab-delimited with coordinate, sample-membership, and genotype fields per merged region
- Per-sample -genotype copy-number values are plausible (near 2 for diploid regions, since copy number = mean regional RD / mean autosomal RD × 2; e.g., README example yields ~1.93–1.95)
- Merged regions are sorted by start coordinate
- Filtered merged calls respect the set thresholds (Q0 ≤ 0.5, size ≥ 100000, optionally p ≤ 0.00001)

## Limitations

- Merging clusters calls only by reciprocal overlap >50%; calls with smaller overlap are not joined
- The more accurate approach for discovering cancer and mosaic CNAs was still under development and the somatic CNA caller is only a prototype, so joint-calling as described is best suited to germline CNVs (e.g., trio data)
- Genotype copy number relies on mean RD normalization to mean autosomal RD scaled by 2 and can be affected by uncorrected GC/mappability bias (mitigate with provided GC/AT 100-bp bin resource data and optional 1000 Genomes P-mask filtering via -rd_use_mask)
- Annotation of merged calls via Ensembl REST API is optional and requires a web connection

## Evidence

- [discussion] Merging calls over multiple samples with reciprocal overlap >50%: "select calls from other samples with reciprocal overlap >50%"
- [abstract] CNV calls can be filtered, annotated, and merged: "CNV calls can be filtered using several criteria, annotated, and merged over multiple samples"
- [discussion] Genotyping copy number definition: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
- [readme] README multi-sample merging command: "> cnvpytor -root file1.pytor file2.pytor ... -view 100000 
print merged_calls"
- [readme] README viewer filter parameters: "set Q0_range 0 0.5
set size_range 100000 inf
set print_filename output.xls"
