---
name: trie-based-sequence-indexing
description: Use when long-read barcode segments extracted downstream of an adapter
  alignment (e.g., scTagger extract_lr_bc 20bp segments) must be matched against up
  to tens of thousands of short-read 10x cellular barcodes allowing up to e=2 edits.
when_to_use_negative:
- When the number of relevant barcodes grows far beyond thousands to tens of thousands
  — scTagger's trie design is not expected to scale well in that regime
- When you need to match UMI sequences between short-read and long-read data — this
  is not supported by scTagger and is planned future work
- When the long-read template lacks a fixed sequence adjacent to the barcode (e.g.,
  non-10x platforms) — support for other platforms is only possible conditionally
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: scTagger
  role: Implements the trie construction (k-mer insertion of selected SR barcodes
    with k = bl + e) and edit-distance-aware querying of 20bp LR segments via the
    match_trie command
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: 'Brute-force dynamic-programming baseline used to validate that trie-based
    matching achieves near-exact edit-distance match rates (e.g., Large dataset: 53.26%
    matches vs FLAMES 30.05%)'
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:4c3c2bc1749719f285c207073012fbcd39901bb5fa46835d78a9b549867229bc
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
  - FLAMES
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# trie-based-sequence-indexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A trie-based data structure that indexes k-mers of selected short-read cellular barcodes and supports edit-distance-aware querying, enabling scTagger's match_trie stage to assign 20bp long-read barcode segments to barcodes within a maximum of e=2 substitutions, insertions, or deletions.

## When to use

Use when long-read barcode segments extracted downstream of an adapter alignment (e.g., scTagger extract_lr_bc 20bp segments) must be matched against up to tens of thousands of short-read 10x cellular barcodes allowing up to e=2 edits.

## When NOT to use

- When the number of relevant barcodes grows far beyond thousands to tens of thousands — scTagger's trie design is not expected to scale well in that regime
- When you need to match UMI sequences between short-read and long-read data — this is not supported by scTagger and is planned future work
- When the long-read template lacks a fixed sequence adjacent to the barcode (e.g., non-10x platforms) — support for other platforms is only possible conditionally

## Inputs

- Long-read segments TSV from extract_lr_bc: read id, best adapter edit distance, adapter match start position, 20bp long-read segment
- Selected short-read barcodes TSV from extract_sr_bc (or extract_sr_bc_from_lr): barcode, number of appearances

## Outputs

- Gzipped TSV with columns: read id, minimum edit distance, number of matching SR barcodes, LR segment, list of minimum-edit-distance barcodes
- Bar plot of the number of long reads by the minimum edit distance of their matched barcode

## How to apply

Run scTagger's match_trie: load the long-read segments TSV from extract_lr_bc (read id, best adapter edit distance, adapter match start, 20bp segment) and the selected short-read barcode TSV from extract_sr_bc. Build the trie by inserting k-mers of length k = bl + e (defaults bl=16, e=2, set via the -mr and -bl flags), and keep a node-depth map for nodes at depth >= bl - e to support backtracking during query. Query each long-read segment recursively against the trie, tracking the minimum unit-cost Levenshtein distance (substitutions, insertions, deletions) up to e=2 and recording all barcodes attaining that minimum; the default e=2 is grounded in the finding that the vast majority of long reads with >2 edits have no unique barcode match. Parallelize trie construction and querying by partitioning k-mers by their prefix of length p (4^p partitions), distributing partitions across threads (-t, default 16) within a memory cap (-m, default 16.0 GB). Emit a gzipped TSV with five columns (read id, minimum edit distance, number of matching SR barcodes, LR segment, list of minimum-edit-distance barcodes) and a bar plot of long reads by minimum edit distance.

## Related tools

- **scTagger** (Implements the trie construction (k-mer insertion of selected SR barcodes with k = bl + e) and edit-distance-aware querying of 20bp LR segments via the match_trie command) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Brute-force dynamic-programming baseline used to validate that trie-based matching achieves near-exact edit-distance match rates (e.g., Large dataset: 53.26% matches vs FLAMES 30.05%)) — https://github.com/Martinsos/edlib

## Examples

```
./scTagger.py match_trie -lr "path/to/output/extract/long-read/segment" -sr "path/to/output/extract/top/short-read" -o "path/to/output/file" -t 16
```

## Evaluation signals

- The gzipped output TSV exists with all five required fields populated: read id, minimum edit distance, number of matching SR barcodes, LR segment, and the list of minimum-edit-distance barcodes
- All reported minimum edit distances in the output are <= the configured maximum of e=2
- Trie k-mer length equals bl + e (default 16 + 2 = 18) and the node-depth map is retained for nodes at depth >= bl - e
- Match/ambiguous/mismatch/skip percentages on the simulated datasets (Figshare 10.6084/m9.figshare.19740475.v1) reproduce Table 3, with scTagger match rate close to the edlib brute-force baseline (Large: 53.26%) and well above FLAMES (30.05%)
- The bar plot of long-read counts by minimum edit distance is generated and its distance axis respects the e=2 cap

## Limitations

- The default maximum edit distance e=2 excludes long reads with >2 edits, most of which have no unique barcode match and are not useful for downstream analysis
- scTagger in its current form is specific for 10x Genomics scRNA-seq; other platforms are supported only if the LR template includes some fixed sequence next to the barcode
- Memory use is a known weakness, and scTagger is not expected to scale well if the number of relevant barcodes becomes much bigger than thousands to tens of thousands

## Evidence
