---
name: long-read-sam-bam-alignment-filtering
description: 'Use when aligned long-read SAM/BAM files must be quality-controlled
  before variant-feature extraction: reads with alignment quality <20 or FLAG 4 (>256)
  must be removed prior to CIGAR-based deletion analysis.'
when_to_use_negative:
- Input is unaligned raw reads (FASTQ) — the skill operates on already-aligned SAM/BAM
  records with MAPQ and FLAG fields.
- The analysis goal is insertion or base substitution calling — the paper states the
  study currently focuses exclusively on deletion variants calling.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: DASV
  role: Pipeline whose preprocessing stage applies this MAPQ >= 20 / FLAG 4 and >256
    filtering to SAM alignments before extracting deletion features from the CIGAR
    field and rendering 100x100 images
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:d326d3ebae44a79aaacfa03f4bc3c5799696f129fdfd488103d7bc2643b4b736
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# long-read-sam-bam-alignment-filtering

## Summary

Quality-control step of the DASV deletion-SV pipeline: third-generation sequencing SAM/BAM alignments (e.g., GIAB AshkenazimTrio HG002) are filtered to drop reads with alignment quality <20 and FLAG identifiers of 4 and >256, so only confidently placed primary alignments feed downstream CIGAR-field deletion-feature extraction.

## When to use

Use when aligned long-read SAM/BAM files must be quality-controlled before variant-feature extraction: reads with alignment quality <20 or FLAG 4 (>256) must be removed prior to CIGAR-based deletion analysis.

## When NOT to use

- Input is unaligned raw reads (FASTQ) — the skill operates on already-aligned SAM/BAM records with MAPQ and FLAG fields.
- The analysis goal is insertion or base substitution calling — the paper states the study currently focuses exclusively on deletion variants calling.

## Inputs

- Aligned SAM/BAM files from third-generation (long-read) sequencing data, e.g. GIAB AshkenazimTrio HG002 alignments
- Filter thresholds: alignment quality number >= 20; excluded FLAG identifiers 4 and >256

## Outputs

- Quality-controlled SAM file containing only reads with alignment quality >= 20 and FLAG not equal to 4 or >256, ready for CIGAR-field deletion-feature extraction

## How to apply

Load the aligned SAM/BAM files for the sample (in DASV, alignments of HG002 from the GIAB AshkenazimTrio deposit). Remove sequenced fragments whose alignment quality number is <20, and drop records with a FLAG identifier of 4 (unmapped) and >256, so that only primary, confidently placed alignments remain. Rationale: the subsequent 'Framing the range of variation' step counts deletions from the CIGAR field of this quality-controlled SAM file and selects high-frequency regions by the file's average sequencing depth, so noisy or non-primary alignments would corrupt deletion counts. Pass the filtered SAM directly to CIGAR-field parsing rather than re-aligning, keeping MAPQ and FLAG semantics intact.

## Related tools

- **DASV** (Pipeline whose preprocessing stage applies this MAPQ >= 20 / FLAG 4 and >256 filtering to SAM alignments before extracting deletion features from the CIGAR field and rendering 100x100 images) — https://github.com/deconvolution-w/DASV

## Evaluation signals

- Every retained record in the output SAM/BAM has an alignment quality number >= 20.
- No retained record carries a FLAG identifier of 4 or a FLAG value >256.
- The filtered file is the input from which deletion features are counted per region via the CIGAR field; if per-region deletion counts or average sequencing depth cannot be computed, filtering was mis-applied.
- Downstream consistency: tiling of selected high-frequency regions (window size 100, stride 50) and rendering into 100x100 pixel PNGs should succeed on the filtered alignments.

## Limitations

- The MAPQ 20 cutoff and FLAG 4/>256 exclusions are fixed in the DASV pipeline; the article does not describe per-dataset tuning of these thresholds.
- Filtering serves deletion-variant calling only; insertion and base substitution detection are explicitly future work.
- The article notes DASV is not yet packaged as user-friendly software, so this step must be reproduced from the DASV GitHub repository.

## Evidence

- [methods] Quality Control Of Gene Files: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] Framing The Range Of Variation: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [abstract] Image conversion step context: "DASV converts gene alignment information into images"
