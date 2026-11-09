---
name: cigar-field-deletion-feature-extraction
description: Use when long-read SAM/BAM alignments must be mined for deletion signatures
  from CIGAR D operators and converted into candidate regions for structural-variant
  calling or image-based feature encoding.
when_to_use_negative:
- The goal includes insertion or base-substitution detection — the article states
  'This study currently focuses exclusively on experiments related to deletion variants
  calling'.
- Input alignments are short NGS reads (e.g. Illumina 250–300bp) rather than third-generation
  long reads, for which this method was designed.
- CIGAR strings are unavailable or unreliable (e.g. only fastq data without an aligned
  SAM/BAM file).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DASV
  role: Implements CIGAR-based deletion feature extraction and high-frequency region
    framing as the first processing stages (QC filtering, 'Framing The Range Of Variation')
    of the deletion SV calling pipeline.
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:d7083c5202f56acf75b4b6f89f2775a894244fd0f6a5b1ae14d7cf44e482fd98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cigar-field-deletion-feature-extraction

## Summary

Extracts deletion-variant features from the CIGAR field of quality-controlled SAM/BAM alignments (GIAB AshkenazimTrio HG002, MAPQ>=20, FLAG 4 and >256 removed) and frames high-frequency variation regions for downstream image rendering in the DASV deletion-calling pipeline.

## When to use

Use when long-read SAM/BAM alignments must be mined for deletion signatures from CIGAR D operators and converted into candidate regions for structural-variant calling or image-based feature encoding.

## When NOT to use

- The goal includes insertion or base-substitution detection — the article states 'This study currently focuses exclusively on experiments related to deletion variants calling'.
- Input alignments are short NGS reads (e.g. Illumina 250–300bp) rather than third-generation long reads, for which this method was designed.
- CIGAR strings are unavailable or unreliable (e.g. only fastq data without an aligned SAM/BAM file).

## Inputs

- Quality-controlled SAM/BAM alignment file (e.g. GIAB AshkenazimTrio HG002, reads with MAPQ <20 and FLAG 4 and >256 already removed)
- File-level average sequencing depth estimate used as the high-frequency region threshold

## Outputs

- High-frequency deletion variation ranges per genomic region (framed variant ranges)
- Per-region 100-base windows tiled with sliding window size 100 and stride 50, ready for sequence encoding and 100x100 image rendering

## How to apply

First apply quality control to the SAM file: drop reads with alignment quality <20 and FLAG identifiers of 4 and >256. Then parse the CIGAR field of the filtered alignments to obtain deletion features, count the number of deletions within the same region, merge ranges within a region, and select high-frequency regions according to the file's average sequencing depth. Tile each selected high-frequency region with a sliding window of size 100 and stride 50 so that downstream steps can render each window onto a 100x100 pixel canvas. This framing step maps deletion-variant features of genes onto corresponding regions of an image, which benefits local perception of data features; the framed ranges also define where the sample genome base sequence is subsequently encoded from the pre-processed BAM file.

## Related tools

- **DASV** (Implements CIGAR-based deletion feature extraction and high-frequency region framing as the first processing stages (QC filtering, 'Framing The Range Of Variation') of the deletion SV calling pipeline.) — https://github.com/deconvolution-w/DASV

## Evaluation signals

- Every retained alignment has alignment quality >=20 and no FLAG value of 4 or >256.
- Extracted deletion features derive exclusively from the CIGAR D field of the quality-controlled SAM file.
- Selected candidate regions are those whose per-region deletion counts exceed the selection threshold derived from the file's average sequencing depth.
- Windows tiled from each high-frequency region use window size 100 and stride 50, yielding bounded windows suitable for the 100-pixel image step.
- Downstream validation: each rendered window image exists and is a 100x100 RGB PNG confined to white background plus CIGAR-mode colors red/green/blue/black as in Fig. 2.

## Limitations

- The 100-sized sliding window can introduce information loss during conversion into images; the article notes this loss is not quantified ('mitigating potential information loss during conversion into images (primarily due to the use of 100-sized sliding windows)').
- High-frequency region selection depends on the file's average sequencing depth, so results vary with coverage (the paper evaluates 90X, 65X, 45X, 20X simulated datasets).
- The repository README contains no usage instructions or concrete invocation, limiting reproducibility of this stage alone.

## Evidence

- [methods] Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [methods] High-frequency selection rationale: "The number of deletions within the same region was counted, and high-frequency regions were selected according to the average sequencing depth of the file"
- [methods] Sliding window parameters: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [intro] Local perception rationale: "Convert gene alignment information into images. This would map the deletion variants features of genes to the corresponding regions of the image"
