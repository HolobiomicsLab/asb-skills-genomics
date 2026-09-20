---
name: transcriptome-to-genome-coordinate-liftover
description: Use when m6A calls are reported in transcript coordinates (e.g. per-transcript
  positions from Nanom6A, Yanocomp, DRUMMER, EpiNano) but downstream benchmarking
  requires genomic positions comparable to a gold-standard peaks BED file.
when_to_use_negative:
- Tool output is already reported in genomic coordinates and only needs thresholding/standardization,
  not coordinate conversion
- The dataset uses a genome other than the one the lift-over script was built for
  (the supplied script targets the SK1 yeast reference), unless the script is adapted
- No gold-standard peaks BED file or genomic-coordinate comparison is planned, e.g.
  when only transcript-level tool outputs are inspected directly
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: NanOlympicsMod
  role: Pipeline hosting the Scripts folder containing the lift-over python script
    and the post-processing R scripts that consume lifted coordinates to build per-tool
    BED files
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine that orchestrates the post-processing steps (including coordinate
    conversion) configured in pipeline.conf
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:e77c57cbafb51d92243f93f855a07460489b23c5ef665406dcd3cab4df8b044f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Transcriptome-to-genome coordinate liftover

## Summary

Converts transcript-level m6A calls produced by the 14 dRNA-seq detection tools in the NanOlympicsMod pipeline into genomic coordinates on a reference genome (SK1 for the yeast dataset), so that heterogeneous tool outputs can be merged into standardized BED files and compared against gold-standard peak sets.

## When to use

Use when m6A calls are reported in transcript coordinates (e.g. per-transcript positions from Nanom6A, Yanocomp, DRUMMER, EpiNano) but downstream benchmarking requires genomic positions comparable to a gold-standard peaks BED file.

## When NOT to use

- Tool output is already reported in genomic coordinates and only needs thresholding/standardization, not coordinate conversion
- The dataset uses a genome other than the one the lift-over script was built for (the supplied script targets the SK1 yeast reference), unless the script is adapted
- No gold-standard peaks BED file or genomic-coordinate comparison is planned, e.g. when only transcript-level tool outputs are inspected directly

## Inputs

- Raw per-tool m6A detection outputs from 14 dRNA-seq tools (transcript-level coordinates)
- Transcriptome annotation / mapping between transcript positions and genomic positions
- SK1 reference genome (yeast) for coordinate projection
- Selected threshold set ('default' or 'relaxed' via --threshold)

## Outputs

- One standardized BED file per tool with genomic position, modification status and numeric confidence value
- Lifted-over m6A peaks aligned to the SK1 reference genome

## How to apply

After running the NanOlympicsMod Nextflow pipeline, parse each tool's raw output accounting for its score type (Nanom6A probability, Yanocomp/DiffErr FDR, DRUMMER p-value, EpiNano sum-of-errors or residuals). For tools reporting transcriptome-level positions, apply the dedicated lift-over python script from the repository's Scripts folder, which was developed to map m6A peaks onto the SK1 reference genome for the yeast dataset. Then, using the post-processing R script designated by --postprocessingScript, write one standardized BED file per tool containing genomic position, modification status and a numeric confidence value under the threshold set selected via --threshold (default or relaxed). Validate that every BED record carries a valid genomic position, a modification status and a numeric confidence column before benchmarking against the m6A gold-standard peaks bed file.

## Related tools

- **NanOlympicsMod** (Pipeline hosting the Scripts folder containing the lift-over python script and the post-processing R scripts that consume lifted coordinates to build per-tool BED files) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine that orchestrates the post-processing steps (including coordinate conversion) configured in pipeline.conf) — https://github.com/mfurla/NanOlympicsMod

## Evaluation signals

- Each per-tool BED file exists and is non-empty after post-processing
- Every BED record carries a valid genomic position, a modification status and a numeric confidence column consistent with the applied threshold set
- Lifted coordinates lie within the bounds of the SK1 reference genome and correspond to the source transcript's genomic span
- Confidence values match the score type declared per tool (e.g. Nanom6A probability vs Yanocomp/DiffErr FDR vs DRUMMER p-value)

## Limitations

- The lift-over script was developed specifically to map m6A peaks to the SK1 reference genome; other genomes require adaptation
- Some benchmarked tools produced empty output files or failed to complete, so the corresponding BED files cannot be generated for all 14 tools
- Transcript-level calls rely on the transcriptome annotation being consistent between the detection tool and the reference genome used for lift-over

## Evidence

- [methods] a python script to lift-over m6A peaks to SK1 reference genome: "a python script to lift-over m6A peaks to SK1 reference genome"
- [readme] Scripts folder contents and repository purpose: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
