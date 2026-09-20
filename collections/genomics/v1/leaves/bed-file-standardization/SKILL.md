---
name: bed-file-standardization
description: Use when raw outputs from multiple Nanopore dRNA-seq m6A detection tools
  with incompatible formats and score types must be converted into a common BED representation
  for benchmarking or cross-tool comparison.
when_to_use_negative:
- When analyzing a single m6A tool whose native output is already in genomic-coordinate
  BED form and does not require cross-tool harmonization.
- When the goal is raw signal processing of FAST5 data (e.g. basecalling or resquiggling)
  rather than post-hoc call standardization.
- When tools failed to complete or produced empty output files — the article reports
  several benchmark runs had to be abandoned for exactly this reason, and standardization
  of empty inputs is meaningless.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline whose post-processing module performs the per-tool conversion
    into a common BED format
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized pipeline, configured via pipeline.conf
    and run with pipeline.nf
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:72e663ff0aefdf1c6e37a3208c8737639a9311427f4bd9973afc5b55f6daac4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bed-file-standardization

## Summary

Standardize heterogeneous m6A detection tool outputs (e.g. Nanom6A probability, Yanocomp/DiffErr FDR, DRUMMER p-value, EpiNano error scores) into a common per-tool BED format with genomic position, modification status and a numeric confidence value, so that the 14 tools benchmarked in NanOlympicsMod can be uniformly compared.

## When to use

Use when raw outputs from multiple Nanopore dRNA-seq m6A detection tools with incompatible formats and score types must be converted into a common BED representation for benchmarking or cross-tool comparison.

## When NOT to use

- When analyzing a single m6A tool whose native output is already in genomic-coordinate BED form and does not require cross-tool harmonization.
- When the goal is raw signal processing of FAST5 data (e.g. basecalling or resquiggling) rather than post-hoc call standardization.
- When tools failed to complete or produced empty output files — the article reports several benchmark runs had to be abandoned for exactly this reason, and standardization of empty inputs is meaningless.

## Inputs

- Raw per-tool m6A detection outputs for the 14 benchmarked tools (Nanom6A, Yanocomp, DiffErr, DRUMMER, EpiNano, ELIGOS, Nanocompore, m6Anet, DENA, etc.)
- Tool-specific score options: --nanom6AP, --yanocompFDR, --differrFDR, --drummerPval, --epinanoErrorSumErr, --epinanoErrorResiduals
- Threshold set selection via --threshold ('default' or 'relaxed')
- Transcriptome-to-genome mapping resources (python lift-over script, SK1 reference genome)
- Post-processing R script path (--postprocessingScript)

## Outputs

- One standardized BED file per tool, with columns for genomic position, modification status and numeric confidence value
- Harmonized per-site calls suitable for comparison against the m6A gold-standard peaks bed file

## How to apply

Load each tool's raw output from the results directory configured in pipeline.conf, and parse it according to its native score type, accounting for the tool-specific score columns exposed via --nanom6AP (probability), --yanocompFDR and --differrFDR (FDR), --drummerPval (p-value), and --epinanoErrorSumErr / --epinanoErrorResiduals (sum-of-errors and residuals). Where calls are transcriptome-level, convert them to genomic coordinates using the dedicated lift-over python script for mapping m6A peaks to the SK1 reference genome. Apply the selected threshold set ('default' or 'relaxed', chosen via --threshold) to assign each site's modification status, then write one standardized BED file per tool using the post-processing R script designated by --postprocessingScript. The rationale is that only after harmonization of score types and coordinate systems can per-tool calls be compared against the m6A gold-standard peaks bed file and against each other within the benchmarking framework.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline whose post-processing module performs the per-tool conversion into a common BED format) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized pipeline, configured via pipeline.conf and run with pipeline.nf) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Each per-tool BED file exists and is non-empty (the article notes some tools 'unexpectedly generate[d] empty output files').
- Every BED record carries a valid genomic position, a modification status, and a numeric confidence column consistent with the applied threshold set ('default' or 'relaxed').
- Transcriptome-level calls that required lift-over map correctly to the SK1 reference genome coordinates.
- The confidence column type matches the tool's native score type (Nanom6A probability, Yanocomp/DiffErr FDR, DRUMMER p-value, EpiNano error sum/residuals).
- Downstream benchmarking (precision/recall against m6A gold-standard peaks) runs without format errors across all 14 tools.

## Limitations

- Several tools could not complete benchmark runs; for a few, runs were abandoned 'due to lack of evident progress in the run or the unexpected generation of empty output files', yielding nothing to standardize.
- Tools lacking a genomic-coordinate native output require the SK1 lift-over script, so coordinate correctness depends on that mapping step.
- Standardized calls inherit each tool's precision/recall preferences and coverage-depth sensitivity; the article stresses performance 'markedly differed across datasets' and with sequencing depth, so threshold choice ('default' vs 'relaxed') materially changes output.

## Evidence

- [readme] R scripts for post-processing and lift-over: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [methods] Threshold set for filtering m6A sites: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [methods] Gold-standard peaks bed input: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [discussion] Tool performance varied across datasets: "The performance of the tools markedly differed across datasets, and methods adopting different approaches showed different preferences in terms of precision and recall."
