---
name: m6a-detection-output-parsing
description: Use when raw outputs from multiple m6A detection tools on Nanopore dRNA-seq
  data have incompatible score types and formats and must be converted to a common
  BED format for cross-tool comparison or benchmarking.
when_to_use_negative:
- When analyzing a single tool's output for its own downstream use, where no cross-tool
  format standardization is needed
- When m6A calls are already in a common, validated BED format with consistent score
  columns
- When the pipeline run itself failed to produce tool outputs (e.g. abandoned runs
  or empty output files) — parse only after confirming non-empty results
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0080
tools:
- name: NanOlympicsMod
  role: Pipeline providing the post-processing R script and configuration options
    that standardize each tool's m6A calls into BED format
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized NanOlympicsMod pipeline including
    the output-parsing step
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:c31032447f9ca62885d6e5f6ab6707d5333a6e5a388664cd3ea05ef299872ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# m6a-detection-output-parsing

## Summary

A post-processing method from the NanOlympicsMod Nextflow pipeline that converts the heterogeneous raw outputs of 14 Nanopore dRNA-seq m6A detection tools (e.g. Nanom6A probabilities, Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano error sums) into standardized per-tool BED files so calls are directly comparable within a single benchmark.

## When to use

Use when raw outputs from multiple m6A detection tools on Nanopore dRNA-seq data have incompatible score types and formats and must be converted to a common BED format for cross-tool comparison or benchmarking.

## When NOT to use

- When analyzing a single tool's output for its own downstream use, where no cross-tool format standardization is needed
- When m6A calls are already in a common, validated BED format with consistent score columns
- When the pipeline run itself failed to produce tool outputs (e.g. abandoned runs or empty output files) — parse only after confirming non-empty results

## Inputs

- Raw output files from each of the 14 m6A detection tools run on dRNA-seq data
- NanOlympicsMod results directory path from pipeline.conf
- Reference genome / transcriptome files (e.g. SK1 yeast strain) for coordinate lift-over
- Threshold set selection ('default' or 'relaxed') via --threshold

## Outputs

- One standardized BED file per tool with genomic position, modification status and numeric confidence value

## How to apply

Load each tool's raw output from the NanOlympicsMod results directory specified in pipeline.conf, and parse each one according to its native score type — Nanom6A probability (--nanom6AP), Yanocomp and DiffErr FDR (--yanocompFDR, --differrFDR), DRUMMER p-value (--drummerPval), and EpiNano sum-of-errors and residuals (--epinanoErrorSumErr, --epinanoErrorResiduals). Where calls are transcriptome-level, convert them to genomic coordinates using the lift-over python script developed for mapping m6A peaks to the SK1 reference genome. Apply the configured significance cut-offs — the 'default' or 'relaxed' threshold set chosen via --threshold — to assign the modification status of each site. Write one standardized BED file per tool with genomic position, modification status and confidence value, using the post-processing R script designated by --postprocessingScript.

## Related tools

- **NanOlympicsMod** (Pipeline providing the post-processing R script and configuration options that standardize each tool's m6A calls into BED format) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized NanOlympicsMod pipeline including the output-parsing step) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Each per-tool BED file exists and is non-empty in the results directory
- Every BED record carries a valid genomic position
- Every BED record carries a modification status column
- Every BED record carries a numeric confidence column consistent with the applied threshold set ('default' or 'relaxed')
- Lift-over of transcriptome-level calls to genomic coordinates is correct with respect to the SK1 reference genome mapping

## Limitations

- Several tools could not complete the benchmark run and produced empty output files, which cannot be standardized
- Transcriptome-level calls require a lift-over step to genomic coordinates, adding a mapping dependency (SK1 reference genome for yeast)
- Score semantics differ per tool (probability, FDR, p-value, error sums), so the 'confidence' column is not comparable in absolute terms across tools without applying the same threshold set

## Evidence

- [full_text] NanOlympicsMod includes a dedicated post-processing module that converts each tool's output into a common format, so that m6A calls from tools with otherwise incompatible output types can be uniformly compared.: [evidence span withheld — anchor status: not_found]
- [methods] --threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed'): "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] a python script to lift-over m6A peaks to SK1 reference genome: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1"
- [discussion] We experienced difficulties completing the run for several tools, and for few of them we had to renounce, due to lack of evident progress in the run or the unexpected generation of empty output files: "for few of them we had to renounce, due to lack of evident progress in the run or the unexpected generation of empty output files"
