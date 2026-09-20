---
name: rna-modification-site-annotation
description: Use when raw per-tool m6A caller outputs (e.g. Nanom6A probabilities,
  Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano error sums) must be converted to
  a common genomic BED format for cross-tool comparison.
when_to_use_negative:
- Input is already a standardized BED file with valid genomic position, modification
  status and confidence columns
- The analysis goal is running the m6A detection tools themselves (basecalling, resquiggling)
  rather than post-processing their outputs
- Benchmarking against orthogonal reference m6A peak sets rather than harmonizing
  raw caller outputs
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline whose post-processing module converts each tool's output
    into a common format via configurable R post-processing scripts
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine orchestrating the containerized post-processing steps defined
    in pipeline.nf and pipeline.conf
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:bf7e7d3134e4ff5862d28284cf4d3960bc0b402952e2e4468afae520f4d90b8e
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
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rna-modification-site-annotation

## Summary

Standardizes heterogeneous raw outputs of 14 m6A detection tools from Nanopore direct RNA sequencing into a single common BED format, so that m6A calls from otherwise incompatible tools can be uniformly compared within the NanOlympicsMod benchmarking pipeline.

## When to use

Use when raw per-tool m6A caller outputs (e.g. Nanom6A probabilities, Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano error sums) must be converted to a common genomic BED format for cross-tool comparison.

## When NOT to use

- Input is already a standardized BED file with valid genomic position, modification status and confidence columns
- The analysis goal is running the m6A detection tools themselves (basecalling, resquiggling) rather than post-processing their outputs
- Benchmarking against orthogonal reference m6A peak sets rather than harmonizing raw caller outputs

## Inputs

- Raw m6A detection tool outputs for the 14 tools (Nanom6A probabilities, Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano sum-of-errors and residuals)
- Transcriptome-to-genome coordinate mappings / lift-over resources for the SK1 reference genome
- Threshold configuration ('default' or 'relaxed' set via --threshold)
- NanOlympicsMod results directory specified via pipeline.conf

## Outputs

- One standardized BED file per m6A detection tool with genomic position, modification status and numeric confidence column

## How to apply

Parse each of the 14 m6A detection tools' raw outputs, accounting for its distinct score type as parameterized by pipeline options (--nanom6AP for probabilities, --yanocompFDR and --differrFDR for FDR, --drummerPval for p-values, --epinanoErrorSumErr and --epinanoErrorResiduals for EpiNano error metrics). Convert transcriptome-level calls to genomic coordinates, using the lift-over python script developed for mapping m6A peaks to the SK1 reference genome where required. Apply the configured significance cut-offs — the 'default' or 'relaxed' threshold set selected via --threshold — to assign each site's modification status. Write one standardized BED file per tool with genomic position, modification status and numeric confidence value, using the post-processing R script designated by --postprocessingScript.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline whose post-processing module converts each tool's output into a common format via configurable R post-processing scripts) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine orchestrating the containerized post-processing steps defined in pipeline.nf and pipeline.conf) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Each per-tool BED file exists and is non-empty
- Every BED record carries a valid genomic position
- Every BED record carries a modification status
- Every BED record carries a numeric confidence column consistent with the applied 'default' or 'relaxed' threshold set
- Scores are interpreted with the correct score type per tool (probability vs FDR vs p-value vs error sum/residuals)

## Limitations

- Lift-over to genomic coordinates requires a dedicated script per reference genome (e.g. the SK1 yeast strain); transcriptome-level calls cannot be compared directly without this conversion
- Tool-specific score types (probability, FDR, p-value, error sums) are not directly comparable without the threshold-set harmonization step
- Several tools could not complete benchmark runs, with empty output files generated unexpectedly, breaking downstream post-processing

## Evidence

- [results] Dedicated post-processing module converts outputs to common format: [evidence span withheld — anchor status: not_found]
- [methods] Threshold filtering option in methods: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] Repository scripts for post-processing and lift-over: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [results] Score-type interpretation per tool: [evidence span withheld — anchor status: not_found]
