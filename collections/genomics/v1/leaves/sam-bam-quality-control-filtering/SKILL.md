---
name: sam-bam-quality-control-filtering
description: Use when starting from aligned third-generation sequencing reads (SAM/BAM)
  for deletion SV calling, and low-MAPQ or multimapped/unaligned fragments must be
  removed before extracting CIGAR-based deletion features.
when_to_use_negative:
- The goal is calling insertion or base-substitution variants — the source study only
  covers deletion variant calling, and the downstream DASV model is deletion-specific
- Input data are already filtered call sets (e.g. VCF deletion calls) rather than
  read-level SAM/BAM alignments — re-filtering has no effect on variant-level output
- Working with short-read NGS data where paired-end mapping or depth-based callers
  (e.g. ReadDepth, BreakDancer) rather than CIGAR-based long-read deletion calling
  is intended
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DASV
  role: Downstream pipeline that consumes the quality-controlled SAM file to frame
    deletion features from CIGAR fields, encode bases, and call deletions with a dual-attention
    multi-scale ResNet
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:629ff71b94440034ca831f7d4b0f78537b153804cd1b378bd6dd1fa2520eb40b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - PBSV
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SAM/BAM quality-control filtering

## Summary

Quality-control step that removes poorly aligned or multimapped sequenced fragments from SAM/BAM alignment files before downstream structural-variant feature extraction. In the DASV deletion-calling pipeline this filtering is the precondition for reliable CIGAR-based deletion feature framing.

## When to use

Use when starting from aligned third-generation sequencing reads (SAM/BAM) for deletion SV calling, and low-MAPQ or multimapped/unaligned fragments must be removed before extracting CIGAR-based deletion features.

## When NOT to use

- The goal is calling insertion or base-substitution variants — the source study only covers deletion variant calling, and the downstream DASV model is deletion-specific
- Input data are already filtered call sets (e.g. VCF deletion calls) rather than read-level SAM/BAM alignments — re-filtering has no effect on variant-level output
- Working with short-read NGS data where paired-end mapping or depth-based callers (e.g. ReadDepth, BreakDancer) rather than CIGAR-based long-read deletion calling is intended

## Inputs

- Aligned SAM/BAM files from third-generation (PacBio CLR/CCS, Oxford Nanopore) sequencing runs
- Alignment metadata fields: MAPQ alignment quality number and FLAG identifier per sequenced fragment

## Outputs

- Quality-controlled SAM file retaining only fragments with MAPQ >= 20 and FLAG = 4 or > 256 excluded
- Filtered alignments suitable for CIGAR-field deletion feature extraction

## How to apply

Apply the filter to the aligned SAM files of long-read samples (e.g. GIAB HG002 CLR/CCS alignments from BioProject PRJNA200694) by removing all sequenced fragments whose alignment quality number (MAPQ) is below 20, and removing fragments with a FLAG identifier of 4 (unaligned) or greater than 256 (secondary/multimapped alignments). The rationale is that low-quality and multimapped fragments corrupt the CIGAR strings from which deletion features are subsequently framed. Only after this filtering are deletion features extracted from the CIGAR field of the quality-controlled SAM file, so any reads retained here feed directly into the DASV image-generation and dual-attention model stages. Decision point: the MAPQ 20 and FLAG 4/>256 cutoffs are the values evaluated in the paper; deviating from them changes the read support statistics that drive the downstream high-frequency-region selection (sliding window 100, stride 50).

## Related tools

- **DASV** (Downstream pipeline that consumes the quality-controlled SAM file to frame deletion features from CIGAR fields, encode bases, and call deletions with a dual-attention multi-scale ResNet) — https://github.com/deconvolution-w/DASV

## Examples

```
samtools view -q 20 -F 4 input.sam | awk 'and($2,256)==0' > quality_controlled.sam
```

## Evaluation signals

- No retained fragment has MAPQ < 20 (maximum dropped-read MAPQ = 19)
- No retained fragment carries FLAG 4 or any FLAG value > 256
- Downstream CIGAR-based deletion feature extraction runs without error on the filtered SAM, confirming the file is parseable and aligned
- F1 score on HG002 real data against the NIST RM 8392 benchmark (5464 deletions >50 bp) should match or approach the paper's Supplementary Table 4 values, with F1 decreasing as coverage depth decreases
- Retained read counts should reflect the expected depth of the input CLR/CCS dataset (e.g. 90X, 65X, 45X, 20X equivalents)

## Limitations

- Flagging fragments with FLAG > 256 removes secondary/supplementary alignments, which may discard legitimate split-read evidence for complex deletions
- The MAPQ 20 threshold was fixed in this study; sensitivity to alternative cutoffs is not reported
- Tool versions and detailed data-availability for reproducing the exact filtered inputs are not stated in the article

## Evidence

- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [readme] Deletion variants calling in third-generation sequencing data based on a dual-attention mechanism: "Deletion variants calling in third-generation sequencing data based on a dual-attention mechanism"
