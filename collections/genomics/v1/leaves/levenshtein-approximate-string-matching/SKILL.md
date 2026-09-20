---
name: levenshtein-approximate-string-matching
description: Use when noisy sequencing data must be matched against a barcode/sequence
  whitelist allowing up to e edits (default e=2), and you need minimum-edit-distance
  matches rather than exact string equality.
when_to_use_negative:
- The LR template has no fixed sequence (e.g. SR adapter) adjacent to the barcode,
  so the 20bp segment cannot be reliably extracted before matching
- The number of relevant barcodes is far larger than thousands to tens of thousands
  — scTagger's current design is not expected to scale well
- An exact-match lookup suffices (e.g. the alternative extract_sr_bc_from_lr strategy
  uses exact matches of 10x Chromium whitelist barcodes on LR segments)
edam_operation: http://edamontology.org/operation_3743
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0654
tools:
- name: scTagger
  role: Implements the trie-based approximate matching of SR barcodes to LR segments
    via its match_trie module with -mr, -bl, -t, and -m parameters
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Sequence-edit-distance library used in the scTagger benchmarking context (v1.3.9)
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:079f03e5a0ec0bf5ceee49dbfe88d06c9a5cdb050d6dc892f04cfbfefd7ae012
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/scTagger
  repo_url: https://github.com/vpc-ccg/scTagger
  tools:
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_3743
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# levenshtein-approximate-string-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Approximate matching of cellular barcode sequences using unit-cost Levenshtein distance (substitutions, insertions, deletions) via an edit-error-aware trie, as implemented in scTagger's match_trie stage for matching SR barcodes to LR barcode segments.

## When to use

Use when noisy sequencing data must be matched against a barcode/sequence whitelist allowing up to e edits (default e=2), and you need minimum-edit-distance matches rather than exact string equality.

## When NOT to use

- The LR template has no fixed sequence (e.g. SR adapter) adjacent to the barcode, so the 20bp segment cannot be reliably extracted before matching
- The number of relevant barcodes is far larger than thousands to tens of thousands — scTagger's current design is not expected to scale well
- An exact-match lookup suffices (e.g. the alternative extract_sr_bc_from_lr strategy uses exact matches of 10x Chromium whitelist barcodes on LR segments)

## Inputs

- Long-read segments TSV from extract_lr_bc: read id, best adapter edit distance, adapter match start, 20bp segment
- Selected short-read barcode list TSV from extract_sr_bc (or extract_sr_bc_from_lr): barcode, number of appearances
- Configured maximum edit distance e (default 2) and barcode length bl (default 16)

## Outputs

- Gzipped TSV with columns: read id, minimum edit distance, number of matching SR barcodes, LR segment, list of minimum-edit-distance barcodes
- Bar plot of the number of long reads by minimum edit distance of their matched barcode

## How to apply

Build a trie over k-mers of length k = bl + e inserted from each query barcode (defaults bl=16, e=2), keeping a node-depth map for nodes at depth >= bl - e to support backtracking during queries. Recursively query each 20bp long-read segment against the trie, tracking the minimum unit-cost Levenshtein distance (substitutions, insertions, deletions) capped at e and recording all barcodes attaining that minimum distance. Parallelize trie construction and querying by partitioning k-mers by prefix of length p (4^p partitions), distributing partitions across threads (-t, default 16) under a memory cap (-m, default 16.0 GB). The e=2 default is grounded in the observation that the vast majority of long reads with >2 edits have no unique barcode match and are not useful downstream.

## Related tools

- **scTagger** (Implements the trie-based approximate matching of SR barcodes to LR segments via its match_trie module with -mr, -bl, -t, and -m parameters) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Sequence-edit-distance library used in the scTagger benchmarking context (v1.3.9)) — https://github.com/Martinsos/edlib

## Examples

```
./scTagger.py match_trie -lr "path/to/output/extract/long-read/segment" -sr "path/to/output/extract/top/short-read" -o "path/to/output/file" -t 16
```

## Evaluation signals

- The gzipped output TSV exists and all five required fields are populated (read id, minimum edit distance, number of matching SR barcodes, LR segment, barcode list)
- No reported minimum edit distance exceeds the configured maximum e=2
- Bar plot shows a right-skewed distribution with most LRs matched at small edit distances; LRs with >2 edits mostly lack unique matches by construction
- Aggregate matching accuracy should be on par with an exact dynamic-programming approach while running orders of magnitude faster

## Limitations

- The vast majority of long reads with more than 2 edits have no unique barcode match, so the default maximum edit distance is e=2
- Memory use is a known weakness; streamlining how inputs are read and processed is planned future work
- The matching does not yet handle UMI sequence comparison between SR and LR data (planned future work)
- Results can be ambiguous: the output records ALL barcodes attaining the minimum distance, not a single best match

## Evidence
