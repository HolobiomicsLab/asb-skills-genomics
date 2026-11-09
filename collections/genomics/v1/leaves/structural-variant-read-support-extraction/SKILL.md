---
name: structural-variant-read-support-extraction
description: Use when a candidate SV list (VCF from the consensus caller) must be
  converted into per-variant image tensors (50,50,3) as CNN input for high-quality
  variant filtering in an ONT long-read pipeline.
when_to_use_negative:
- You only need per-variant read counts or genotype confidence scores rather than
  spatial alignment-feature images for CNN filtering
- Input reads are short-read (Illumina) alignments, since this encoding is built around
  long-read ONT alignments and split reads
- You lack the alignment BAM and only hold VCF summaries — the encoding requires per-read,
  per-position alignment features
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0102
tools:
- name: ConsensuSV-ONT
  role: Produces the consensus VCF of candidate SVs that this skill encodes into images
    for CNN-based filtering
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: pysam
  role: Fetches quality-passing read alignments (including split reads) overlapping
    the variant-extended window from the BAM
- name: OpenCV
  role: Resizes and normalizes the raw read-by-position matrix to the 50x50x3 CNN
    input shape
- name: Nextflow
  role: Wraps the encoding step in the pipeline executed with parallel computing inside
    the Docker image
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:2f1b73341114baebbb160acf3d00305ef70bc450d9c28bfbb6a0a8aa565bafef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  repo_url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - Docker
  - Nextflow
  - OpenCV
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# structural-variant-read-support-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes each consensus structural variant candidate from ConsensuSV-ONT into a fixed 50x50x3 image by extracting quality-passing long-read alignments around the variant and resizing the read-by-position matrix, so the CNN filter can score high-quality variants.

## When to use

Use when a candidate SV list (VCF from the consensus caller) must be converted into per-variant image tensors (50,50,3) as CNN input for high-quality variant filtering in an ONT long-read pipeline.

## When NOT to use

- You only need per-variant read counts or genotype confidence scores rather than spatial alignment-feature images for CNN filtering
- Input reads are short-read (Illumina) alignments, since this encoding is built around long-read ONT alignments and split reads
- You lack the alignment BAM and only hold VCF summaries — the encoding requires per-read, per-position alignment features

## Inputs

- VCF of consensus structural variant candidates from ConsensuSV-ONT (chromosome, coordinates, SV type)
- minimap2 alignment BAM of Oxford Nanopore long reads (with split-read records)
- Candidate variant type used to size the flanking window expansion

## Outputs

- One 50x50x3 normalized image (PNG or NPZ array) per candidate variant, named by variant identifier
- Raw read-by-position alignment feature matrices prior to resizing

## How to apply

Parse the consensus VCF to obtain each candidate's coordinates and variant type. For each candidate, use pysam to pull quality-passing reads (including split-read records) overlapping the variant region from the minimap2 BAM (Chaisson-style alignment), expanding the window by the size of the mutation on both sides to capture surrounding context. Build a raw matrix where rows are reads, columns are genomic positions, and cell values encode per-position alignment features. Resize and normalize the matrix with OpenCV into the fixed 50x50x3 shape expected by the first Conv3D layer of the CNN. Write one PNG or NPZ per variant, named by its variant identifier.

## Related tools

- **ConsensuSV-ONT** (Produces the consensus VCF of candidate SVs that this skill encodes into images for CNN-based filtering) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **pysam** (Fetches quality-passing read alignments (including split reads) overlapping the variant-extended window from the BAM)
- **OpenCV** (Resizes and normalizes the raw read-by-position matrix to the 50x50x3 CNN input shape)
- **Nextflow** (Wraps the encoding step in the pipeline executed with parallel computing inside the Docker image) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
import pysam, cv2; aln = pysam.AlignmentFile("aln.bam","rb"); reads = [r for r in aln.fetch(chrom, start-svlen, end+svlen) if r.mapping_quality > 0 and not r.is_supplementary]; img = cv2.resize(matrix, (50, 50, 3))
```

## Evaluation signals

- Output directory contains exactly one 50x50x3 image per candidate; the image count equals the candidate list length from the consensus VCF
- Every saved array/PNG loads with expected shape (50, 50, 3) and expected format (PNG or NPZ)
- The extracted window for each candidate spans the variant coordinates plus flanks of the mutation size on both sides
- Downstream Conv3D layer (None, 50, 50, 3, 4) accepts every encoded tensor without shape errors
- Each output filename maps unambiguously to a variant identifier from the candidate VCF

## Limitations

- No version numbers, parameters, or detailed implementation settings for the encoding are given in the article's discussion section
- The paper notes a need for more detailed comparative analysis across tools and sequencing methods, so generalization of this encoding beyond ONT data is unvalidated
- No changelog is available in the repository, complicating tracking of encoding implementation changes

## Evidence

- [other] Window expansion by mutation size: [evidence span withheld — anchor status: not_found]
