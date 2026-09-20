---
name: edit-distance-sequence-scoring
description: Use when long reads from hybrid LR-SR scRNA-seq (e.g., 10x Genomics split-pool
  libraries) contain a fixed adapter sequence adjacent to the cellular barcode and
  you must locate and score that locus per read by edit distance.
when_to_use_negative:
- The long-read template lacks a fixed sequence adjacent to the barcode (non-10x platforms)
  — scTagger support is conditional on a fixed sequence next to the barcode
- Barcode segments are already extracted and you only need to build the barcode whitelist
  or match barcodes (use extract_sr_bc / extract_sr_bc_from_lr / match_trie instead)
- You need to match UMI sequences between SR and LR data — that is not supported by
  this method
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: edlib
  role: Computes edit-distance-based glocal alignment of the fixed short-read adapter
    sequence against each long read (v1.3.9 used in the study)
  repo: https://github.com/Martinsos/edlib
- name: scTagger
  role: 'Implements the extract_lr_bc stage: runs the adapter scoring, peak-range
    filtering, and 20 bp segment extraction'
  repo: https://github.com/vpc-ccg/scTagger/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:0531abc9767070d77d3831f8635a4badd35aef9915b30c72db64478e81c5f01f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/scTagger
  repo_url: https://github.com/Martinsos/edlib
  tools:
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# edit-distance-sequence-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores approximate occurrences of a fixed short sequence (the Illumina adapter CTACACGACGCTCTTCCGATCT) against every long read by edit distance, using edlib glocal alignment to locate the barcode-adjacent segment in hybrid scRNA-seq long reads. This is the scTagger extract_lr_bc stage that precedes trie-based barcode matching.

## When to use

Use when long reads from hybrid LR-SR scRNA-seq (e.g., 10x Genomics split-pool libraries) contain a fixed adapter sequence adjacent to the cellular barcode and you must locate and score that locus per read by edit distance.

## When NOT to use

- The long-read template lacks a fixed sequence adjacent to the barcode (non-10x platforms) — scTagger support is conditional on a fixed sequence next to the barcode
- Barcode segments are already extracted and you only need to build the barcode whitelist or match barcodes (use extract_sr_bc / extract_sr_bc_from_lr / match_trie instead)
- You need to match UMI sequences between SR and LR data — that is not supported by this method

## Inputs

- Long-read FASTQ files (space-separated paths; gzipped inputs auto-detected by .gz suffix, else -z)
- Optional adapter sequence via -sa (default CTACACGACGCTCTTCCGATCT)
- Optional manual adapter-location ranges via -g (default: detected from data)

## Outputs

- Gzipped TSV with columns: read-id, best edit distance with the short-read adapter, starting point of the long read matching the adapter, extracted long-read segment (default 20 bp)
- Optional plot of optimal adapter alignment locations per read (via -p)

## How to apply

Glocal-align the fixed short-read adapter (default CTACACGACGCTCTTCCGATCT, parameter -sa) to every long read with edlib (v1.3.9), recording the best edit distance per read. Because adapter alignments form peaks on the forward and reverse strands, automatically detect the ranges around these peaks and reject any adapter alignment falling outside them (optionally restricted with -g). For each surviving alignment, extract the segment of --num-bp-afte bases (default 20) starting at the end of the adapter alignment, together with the read-id, best edit distance, and alignment start. The rationale is that the region immediately after the adapter alignment has a higher probability of containing the cellular barcode than other regions of the read; downstream barcode matching is capped at edit distance e=2 because most LRs with >2 edits lack a unique barcode match.

## Related tools

- **edlib** (Computes edit-distance-based glocal alignment of the fixed short-read adapter sequence against each long read (v1.3.9 used in the study)) — https://github.com/Martinsos/edlib
- **scTagger** (Implements the extract_lr_bc stage: runs the adapter scoring, peak-range filtering, and 20 bp segment extraction) — https://github.com/vpc-ccg/scTagger/

## Examples

```
./scTagger.py extract_lr_bc -r "path/to/long/read/fastq" -o "path/to/output/file" -p "path/to/output/plots"
```

## Evaluation signals

- Output TSV contains exactly the four required columns: read-id, best edit distance, adapter alignment start, extracted segment
- Edit distances are non-negative integers, and alignment start positions fall within the automatically detected peak ranges on the forward/reverse strands
- Extracted segments are --num-bp-afte bases long (20 by default) and begin immediately at the end of the adapter alignment
- The optional alignment-location plot shows clear peaks per strand; alignments outside detected ranges are absent from the TSV
- Only reads whose adapter alignments passed the peak-range filter appear in the output TSV

## Limitations

- scTagger in its current form is specific for 10x Genomics scRNA-seq; other platforms are supported only if the LR template includes a fixed sequence next to the barcode
- Adapter alignment ranges must be detectable from the data unless supplied manually with -g; misdetected peaks would wrongly reject valid alignments
- Memory use is a known weakness of the tool (streamlined input handling is planned future work)
- The step scores adapter presence, not barcode identity — exact barcode matching is deferred to the trie stage with default maximum edit distance e=2

## Evidence
