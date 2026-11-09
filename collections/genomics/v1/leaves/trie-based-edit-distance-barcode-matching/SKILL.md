---
name: trie-based-edit-distance-barcode-matching
description: Use when you have LR barcode segments (from extract_lr_bc) and top SR
  barcodes (from extract_sr_bc) for hybrid long-read/short-read scRNA-seq data and
  need to assign long reads to cells under edit errors up to e=2, faster than brute-force
  dynamic programming or FLAMES.
when_to_use_negative:
- When the number of relevant barcodes is expected to be much larger than thousands
  to tens of thousands — scTagger's current trie design is not expected to scale well
  beyond that
- When LR reads require maximum edit distance >2 — most LRs with >2 edits have no
  unique barcode match, so higher e thresholds are uninformative
- When long-reads come from non-10x Genomics scRNA-seq platforms without a fixed sequence
  adjacent to the barcode — scTagger is specific for 10x Genomics scRNA-seq in its
  current form
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scTagger
  role: Provides the match_trie stage that implements the edit-distance-aware trie
    matching between LR segments and SR barcodes
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib (v1.3.9)
  role: Powers the exact brute-force dynamic-programming matching baseline used for
    accuracy and runtime comparison
  repo: https://github.com/Martinsos/edlib
- name: 'FLAMES (commit #18fb83c)'
  role: State-of-the-art comparator tool for LR-barcode association runtime and accuracy
  repo: https://github.com/LuyiTian/FLAMES
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:17d80163f97265e95c64a5cded5dd463fb008fe06fc133079163d160ea60fea1
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
  - 'FLAMES (commit #18fb83c)'
  - Snakemake
  - edlib
  - edlib (v1.3.9)
  - scTagger
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# trie-based-edit-distance-barcode-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches long-read cellular barcode segments to top short-read barcodes using a modified trie data structure that permits nonzero edit-distance matching (default e=2), enabling efficient LR-SR barcode assignment in hybrid single-cell RNA-seq experiments as implemented in scTagger's match_trie stage.

## When to use

Use when you have LR barcode segments (from extract_lr_bc) and top SR barcodes (from extract_sr_bc) for hybrid long-read/short-read scRNA-seq data and need to assign long reads to cells under edit errors up to e=2, faster than brute-force dynamic programming or FLAMES.

## When NOT to use

- When the number of relevant barcodes is expected to be much larger than thousands to tens of thousands — scTagger's current trie design is not expected to scale well beyond that
- When LR reads require maximum edit distance >2 — most LRs with >2 edits have no unique barcode match, so higher e thresholds are uninformative
- When long-reads come from non-10x Genomics scRNA-seq platforms without a fixed sequence adjacent to the barcode — scTagger is specific for 10x Genomics scRNA-seq in its current form

## Inputs

- LR barcode-segment TSV from scTagger extract_lr_bc (read-id, best edit distance, adapter alignment start, 20bp long-read segment)
- Top SR barcode TSV from scTagger extract_sr_bc (barcode, read count)
- Threads (-t, default 16) and memory limit (-m, default 16.0 GB) settings

## Outputs

- Matched barcode assignment per long read (LR-to-barcode matches TSV)
- Per-run timing and peak memory measurements for runtime/memory comparison (e.g., table4_runtime_memory_benchmark.csv)

## How to apply

Run scTagger's match_trie stage on the LR segment TSV (read-id, best adapter edit distance, adapter alignment start, 20bp segment) and the top SR barcode TSV, constructing a trie of SR barcodes modified to return the barcode requiring the least number of edits (substitutions, deletions, insertions) up to the default maximum edit distance e=2, since the vast majority of LRs with >2 edits have no unique barcode match. Parallelize by partitioning the k-mers by their prefix of length p (4^p partitions), controlling threads with -t (default 16) and memory with -m (default 16.0 GB). Record real time, user time, and peak memory per run when benchmarking against the edlib-based brute-force baseline (e<=2) and FLAMES (commit #18fb83c) on the same inputs. Validate that runtime is orders of magnitude lower than the brute-force baseline and FLAMES while accuracy remains on par with the exact dynamic-programming approach.

## Related tools

- **scTagger** (Provides the match_trie stage that implements the edit-distance-aware trie matching between LR segments and SR barcodes) — https://github.com/vpc-ccg/scTagger/
- **edlib (v1.3.9)** (Powers the exact brute-force dynamic-programming matching baseline used for accuracy and runtime comparison) — https://github.com/Martinsos/edlib
- **FLAMES (commit #18fb83c)** (State-of-the-art comparator tool for LR-barcode association runtime and accuracy) — https://github.com/LuyiTian/FLAMES

## Examples

```
./scTagger.py match_trie -lr lr_segments.tsv -sr top_sr_barcodes.tsv -o matched_barcodes.tsv -t 16
```

## Evaluation signals

- scTagger's real/user runtime is orders of magnitude lower than the edlib-based brute-force baseline and FLAMES on all three real datasets (N, NOA1, NOA2)
- Matching accuracy is on par with the exact dynamic-programming matching approach despite the nonzero-edit-distance trie approximation
- Trie matches respect the default maximum edit distance e=2; LRs requiring >2 edits are not uniquely matched
- Peak memory stays within the configured limit (-m 16.0 GB default)
- Per-partition parallelization (4^p prefix partitions) yields speedup consistent with -t thread count

## Limitations

- Memory use is a known weakness; streamlining input reading/processing is planned future work
- scTagger in its current design is specific for 10x Genomics scRNA-seq and not expected to scale well if the number of relevant barcodes grows much beyond thousands to tens of thousands
- UMI sequence matching between SR and LR data is not yet supported

## Evidence
