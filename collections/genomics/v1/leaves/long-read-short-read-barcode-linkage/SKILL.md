---
name: long-read-short-read-barcode-linkage
description: Use when a hybrid scRNA-seq experiment splits one barcoded library into
  Illumina SR and ONT/PacBio LR pools, and LRs must be assigned to cellular barcodes
  shared with SRs; especially when existing tools (FLAMES, SiCeLoRe) are too slow
  for many reads/cells.
when_to_use_negative:
- When UMI-level matching between SR and LR data is required — scTagger does not yet
  support matching unique molecular identifier sequences
- For non-10x scRNA-seq platforms unless the LR template contains a fixed sequence
  next to the barcode; scTagger is specific for 10x Genomics scRNA-seq
- When the relevant barcode set greatly exceeds thousands to tens of thousands of
  barcodes — the current trie design is not expected to scale beyond that range
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: scTagger
  role: 'Performs all three stages: adapter-guided LR segment extraction, SR barcode
    selection from the Cell Ranger BAM (or LR whitelist), and trie-based edit-distance-aware
    barcode matching'
  repo: https://github.com/vpc-ccg/scTagger/
- name: 10x Genomics Cell Ranger
  role: Prepares the input BAM by genomic mapping, filtering, barcode counting, and
    UMI counting from which extract_sr_bc selects the top barcodes
- name: edlib
  role: Brute-force dynamic-programming baseline (v1.3.9) used to compute exact edit-distance
    assignments for validating scTagger's match rate
  repo: https://github.com/Martinsos/edlib
- name: FLAMES
  role: State-of-the-art barcode-matching tool compared against scTagger for accuracy
    and time-efficiency benchmarks
  repo: https://github.com/LuyiTian/FLAMES
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:39a51dd675b7fff1764884b43e1da67c91e37f5e22f39e49df50d46142688f2a
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
  - 10x Genomics Cell Ranger
  - FLAMES
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-short-read-barcode-linkage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches cellular barcodes between long-read (LR) and short-read (SR) single-cell RNA-seq data using scTagger's trie-based matcher, which assigns each LR barcode segment to the SR barcode reachable with the fewest edits (substitutions, insertions, deletions) up to a default maximum edit distance of e=2. This enables relating SR-derived gene expression to LR-derived splicing at the cell level in hybrid 10x Chromium LR-SR experiments.

## When to use

Use when a hybrid scRNA-seq experiment splits one barcoded library into Illumina SR and ONT/PacBio LR pools, and LRs must be assigned to cellular barcodes shared with SRs; especially when existing tools (FLAMES, SiCeLoRe) are too slow for many reads/cells.

## When NOT to use

- When UMI-level matching between SR and LR data is required — scTagger does not yet support matching unique molecular identifier sequences
- For non-10x scRNA-seq platforms unless the LR template contains a fixed sequence next to the barcode; scTagger is specific for 10x Genomics scRNA-seq
- When the relevant barcode set greatly exceeds thousands to tens of thousands of barcodes — the current trie design is not expected to scale beyond that range

## Inputs

- Long-read FASTQ file(s), gzipped or plain (input to extract_lr_bc via -r)
- Cell Ranger BAM of short reads (input to extract_sr_bc via -i)
- 10x Genomics cellular barcode whitelist, e.g. 3M-february-2018.txt.gz (for extract_sr_bc_from_lr via -wl)
- extract_lr_bc output TSV (read id, best adapter edit distance, adapter match start, 20 bp segment)
- extract_sr_bc output TSV (barcode, number of appearances)

## Outputs

- Gzipped TSV of barcode matches with five columns: read id, minimum edit distance, number of matching SR barcodes, LR segment, and list of minimum-edit-distance barcodes
- Bar plot of the number of long reads by minimum edit distance of their matched barcode
- Long-read segment TSV (read id, best adapter edit distance, adapter match start position, 20 bp segment)
- Selected short-read barcode TSV (barcode, count) with cumulative SR coverage plot

## How to apply

Run the three-stage scTagger pipeline. (1) extract_lr_bc glocally aligns the SR adapter (default CTACACGACGCTCTTCCGATCT) to each LR FASTQ read, rejects alignments outside automatically detected peak ranges, and extracts the 20 bp segment following the adapter alignment, recording the best adapter edit distance. (2) extract_sr_bc selects the top barcodes from the Cell Ranger BAM: barcodes are added in 1,000-barcode increments, stopping when an increment adds less than t=0.5% of total SRs (max 25,000 barcodes); alternatively extract_sr_bc_from_lr builds the whitelist by exact-matching the 10x 3M-february-2018 whitelist against LR segments. (3) match_trie inserts k-mers of length k=bl+e (bl=16, e=2) from selected SR barcodes into a trie with a node-depth map for depth >= bl-e to support backtracking, then recursively queries each LR segment tracking unit-cost Levenshtein distance up to e=2, recording all barcodes at the minimum distance; k-mers are partitioned by prefix of length p (4^p partitions) and distributed across threads (-t, default 16) under a memory cap (-m, default 16.0 GB). The rationale for e=2 is that the vast majority of LRs with >2 edits have no unique barcode match, so deeper error tolerance would add cost without useful assignments.

## Related tools

- **scTagger** (Performs all three stages: adapter-guided LR segment extraction, SR barcode selection from the Cell Ranger BAM (or LR whitelist), and trie-based edit-distance-aware barcode matching) — https://github.com/vpc-ccg/scTagger/
- **10x Genomics Cell Ranger** (Prepares the input BAM by genomic mapping, filtering, barcode counting, and UMI counting from which extract_sr_bc selects the top barcodes)
- **edlib** (Brute-force dynamic-programming baseline (v1.3.9) used to compute exact edit-distance assignments for validating scTagger's match rate) — https://github.com/Martinsos/edlib
- **FLAMES** (State-of-the-art barcode-matching tool compared against scTagger for accuracy and time-efficiency benchmarks) — https://github.com/LuyiTian/FLAMES

## Examples

```
./scTagger.py match_trie -lr "path/to/output/extract/long-read/segment" -sr "path/to/output/extract/top/short-read" -o "path/to/output/file" -t "number of threads"
```

## Evaluation signals

- Output gzipped TSV exists with all five required fields populated (read id, minimum edit distance, matching-barcode count, LR segment, matching barcodes)
- All reported minimum edit distances are <= the configured maximum of e=2
- Per-method percentages of match (unique barcode), ambiguous (multiple barcodes at minimum edit distance), mismatch, and skip reproduce Table 3 on simulated datasets — e.g., scTagger Large: 53.26% matches vs FLAMES 30.05%, and near-parity with the edlib brute-force baseline
- Extract_sr_bc stop rule holds: retaining stopped once an increment of 1,000 barcodes adds less than t=0.5% of total SRs, and at most 25,000 barcodes are kept
- A bar plot of long-read counts by minimum matched edit distance is generated for sanity-checking the edit-distance distribution

## Limitations

- scTagger in its current form is specific for 10x Genomics scRNA-seq; other platforms are only supportable if the LR template includes a fixed sequence next to the barcode
- Current design is not expected to scale well if the number of relevant barcodes becomes much bigger than thousands to tens of thousands
- Memory use is a known weakness; streamlined input reading/processing is planned future work
- LRs requiring more than 2 edits mostly have no unique barcode match and are not useful downstream, which caps the recoverable match rate
- No changelog is available for the synthesized source (reproducibility gap)

## Evidence
