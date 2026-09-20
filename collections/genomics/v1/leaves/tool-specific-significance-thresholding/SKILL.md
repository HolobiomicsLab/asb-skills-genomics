---
name: tool-specific-significance-thresholding
description: Use when outputs from multiple m6A detection tools on Nanopore dRNA-seq
  data have incompatible score types and formats and must be thresholded and converted
  into a common BED format for direct benchmarking comparison.
when_to_use_negative:
- Input outputs are already standardized, non-heterogeneous BED files with valid confidence
  columns — re-thresholding would double-apply cut-offs
- Single-tool analysis with no need for cross-tool comparison, where converting to
  the common BED schema adds no benchmarking value
- Tools that failed to run or produced empty output files, which the article reports
  for several tools — thresholds cannot be meaningfully applied to absent calls
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Pipeline hosting the post-processing module that parses per-tool scores, applies
    --threshold cut-off sets, and emits standardized BED files via --postprocessingScript
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized post-processing steps configured
    through pipeline.conf
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:b36fe89ad87434f69d0fae7e745da7df1180dcfa8767eb300da1fe93ccc839e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Tool-specific significance thresholding

## Summary

A post-processing procedure from the NanOlympicsMod pipeline that parses the heterogeneous raw outputs of 14 m6A detection tools (e.g. Nanom6A probabilities, Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano error statistics), converts them to genomic coordinates, and applies per-tool 'default' or 'relaxed' significance cut-offs to write standardized BED files of modification calls.

## When to use

Use when outputs from multiple m6A detection tools on Nanopore dRNA-seq data have incompatible score types and formats and must be thresholded and converted into a common BED format for direct benchmarking comparison.

## When NOT to use

- Input outputs are already standardized, non-heterogeneous BED files with valid confidence columns — re-thresholding would double-apply cut-offs
- Single-tool analysis with no need for cross-tool comparison, where converting to the common BED schema adds no benchmarking value
- Tools that failed to run or produced empty output files, which the article reports for several tools — thresholds cannot be meaningfully applied to absent calls

## Inputs

- Raw per-tool m6A detection outputs from 14 tools (Nanom6A probability scores, Yanocomp/DiffErr FDR values, DRUMMER p-values, EpiNano sum-of-errors and residuals)
- pipeline.conf Nextflow configuration with resultsDir and tool-specific score options
- Transcriptome-to-genome mappings / reference annotations (e.g. SK1 yeast reference genome for lift-over)

## Outputs

- One standardized BED file per m6A detection tool with genomic position, modification status and numeric confidence value
- Modification status per site assigned under the chosen 'default' or 'relaxed' threshold set

## How to apply

Load each tool's raw output from the NanOlympicsMod results directory specified in pipeline.conf and identify its score type, which differs per tool: Nanom6A probability, Yanocomp and DiffErr FDR, DRUMMER p-value, EpiNano sum-of-errors or residuals (controlled by --nanom6AP, --yanocompFDR, --differrFDR, --drummerPval, --epinanoErrorSumErr and --epinanoErrorResiduals options). Where needed, convert transcriptome-level calls to genomic coordinates using the lift-over python script developed for mapping m6A peaks to the SK1 reference genome. Apply the significance cut-off set selected via --threshold ('default' or 'relaxed') to assign the modification status of each site, since the article shows changing stringency cut-offs tunes the precision-recall trade-off. Finally, run the post-processing R script designated by --postprocessingScript to write one standardized BED file per tool containing genomic position, modification status and a numeric confidence value.

## Related tools

- **NanOlympicsMod** (Pipeline hosting the post-processing module that parses per-tool scores, applies --threshold cut-off sets, and emits standardized BED files via --postprocessingScript) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized post-processing steps configured through pipeline.conf) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir" --threshold="default"
```

## Evaluation signals

- Each per-tool BED file exists and is non-empty (the article reports empty output files as a known failure mode)
- Every BED record carries a valid genomic position, a modification status, and a numeric confidence column
- Confidence values are consistent with the applied threshold set ('default' vs 'relaxed') and the tool's native score type (probability, FDR, or p-value)
- Transcriptome-level calls correctly lift over to genomic coordinates on the SK1 reference genome where the lift-over script was required

## Limitations

- Changing the stringency cut-offs tunes the precision-recall trade-off, so 'default' vs 'relaxed' choice strongly affects downstream benchmark results and must be reported
- Several tools could not complete benchmark runs or unexpectedly generated empty output files, leaving no records to threshold
- Score semantics differ across tools (probability vs FDR vs p-value vs error sums), so a single numeric threshold is not comparable across score types without per-tool parameterization

## Evidence

- [other] NanOlympicsMod includes a dedicated post-processing module that converts each tool's output into a common format, so that m6A calls from tools with otherwise incompatible output types can be uniformly compared.: [evidence span withheld — anchor status: not_found]
- [methods] --threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed'): "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [abstract] Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences.: "Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences."
- [readme] Scripts: folder containing ... a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
