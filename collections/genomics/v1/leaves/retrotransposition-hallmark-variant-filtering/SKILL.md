---
name: retrotransposition-hallmark-variant-filtering
description: Use when a merged insertion callset (e.g., cuteSV+Sniffles2 on nanopore
  data) needs specificity improvement by checking reassembled inserted sequences for
  retrotransposition hallmarks before accepting calls.
when_to_use_negative:
- Input calls are deletions or non-insertion SV classes rather than reassembled insertion
  sequences
- The analysis goal is detection/enrichment/genotyping of TEs rather than call-set
  specificity refinement (RetroInspector performs those as separate pipeline stages)
- No reassembled inserted sequences are available (e.g., calls from short-read tools
  like MELT without sequence reassembly)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: RetroInspector
  role: Pipeline that produces the reassembled inserted sequences and supporting-read
    filtered callset on which the hallmark filter is applied
  repo: https://github.com/javiercguard/retroinspector
- name: RapidFuzz
  role: Fuzzy string matching (initial similarity ratio 60%) used to locate the L1
    endonuclease consensus motif TTTTT/AA and its reverse complement on both strands
- name: cuteSV
  role: Variant caller whose insertion calls, merged with Sniffles2, form the callset
    being hallmark-filtered
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Variant caller whose insertion calls, merged with cuteSV, form the callset
    being hallmark-filtered
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: pROC
  role: Used to evaluate precision/recall of the TE insertion callset with and without
    the hallmark filters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:db6beb50d924b3e0ce94dcabaade11b31d65cd0535eb56b7f9552e847ab1edd9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - RapidFuzz
  - RetroInspector
  - Sniffles2
  - cuteSV
  - pROC
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# retrotransposition-hallmark-variant-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters TE insertion calls from reassembled long-read sequences by requiring retrotransposition hallmarks — terminal polyA/polyT tails and the L1 endonuclease target motif TTTTT/AA — improving precision of the RetroInspector cuteSV-Sniffles2 callset to ~90% at 5 and 7 supporting reads.

## When to use

Use when a merged insertion callset (e.g., cuteSV+Sniffles2 on nanopore data) needs specificity improvement by checking reassembled inserted sequences for retrotransposition hallmarks before accepting calls.

## When NOT to use

- Input calls are deletions or non-insertion SV classes rather than reassembled insertion sequences
- The analysis goal is detection/enrichment/genotyping of TEs rather than call-set specificity refinement (RetroInspector performs those as separate pipeline stages)
- No reassembled inserted sequences are available (e.g., calls from short-read tools like MELT without sequence reassembly)

## Inputs

- Reassembled inserted sequences from the cuteSV-Sniffles2 merged insertion callset (RetroInspector reassembly step)
- GRCh38 reference sequence ±20 bp around each insertion site
- Insertion call metadata including insertion coordinates, strand, and supporting-read counts

## Outputs

- Per-insertion annotation of L1 endonuclease motif hit (TTTTT/AA or TT/AAAAA, closest to insertion coordinates)
- Per-insertion polyA/polyT tail annotation (tail position, within terminal 10% of sequence)
- Precision/recall metrics at 5 and 7 supporting-read thresholds with and without hallmark filtering (precision 90.38% and 90.94% reported)
- χ-squared test results comparing hallmark feature proportions in true vs false positives

## How to apply

Retrieve each reassembled inserted sequence from the cuteSV-Sniffles2 merged callset plus the GRCh38 reference ±20 bp around each insertion site. Run a fuzzy text search with RapidFuzz at an initial similarity ratio of 60% for the L1 endonuclease consensus motif TTTTT/AA and its reverse complement TT/AAAAA on both strands, keeping the hit closest to the insertion coordinates when multiple matches occur. Detect polyA/polyT tails by recovering adenine/thymine runs, merging candidates separated by ≤2 bp gaps, selecting the candidate closest to the sequence end (or start for the opposite strand), and requiring the tail to start within the last (or first) 10% of the inserted sequence. Recompute precision and recall with and without the hallmark filters at supporting-read thresholds of 5 and 7 reads, and test true-vs-false-positive hallmark proportions with a χ-squared test.

## Related tools

- **RetroInspector** (Pipeline that produces the reassembled inserted sequences and supporting-read filtered callset on which the hallmark filter is applied) — https://github.com/javiercguard/retroinspector
- **RapidFuzz** (Fuzzy string matching (initial similarity ratio 60%) used to locate the L1 endonuclease consensus motif TTTTT/AA and its reverse complement on both strands)
- **cuteSV** (Variant caller whose insertion calls, merged with Sniffles2, form the callset being hallmark-filtered) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Variant caller whose insertion calls, merged with cuteSV, form the callset being hallmark-filtered) — https://github.com/fritzsedlazeck/Sniffles
- **pROC** (Used to evaluate precision/recall of the TE insertion callset with and without the hallmark filters)

## Examples

```
from rapidfuzz import fuzz; hit = max(candidates, key=lambda c: fuzz.ratio(motif, c.seq)) if fuzz.ratio('TTTTT/AA', best.seq) >= 60 else None
```

## Evaluation signals

- Precision rises above 90% at both 5 and 7 supporting reads (reported: 90.38% and 90.94%) with acceptable recall loss
- PolyA/polyT tail candidate starts within the last (or first) 10% of the inserted sequence and gap-merged candidates are ≤2 bp apart
- When multiple motif matches occur, the kept RapidFuzz hit (≥60% similarity) is the one closest to the insertion coordinates
- χ-squared test on hallmark proportions between true and false positives yields a significant difference (paper reported χ2, p < 2e−16 for hallmark discrimination)
- Baseline strict cuteSV & Sniffles2 performance (precision 80.90% TE / 85.63% all, recall 87.10% TE / 16.34% all, F1 83.89%) is reproduced before filtering

## Limitations

- HGSVC truth-set data was basecalled with an older basecaller, so reassembled sequences may be imperfect, particularly for homopolymers — directly affecting polyA tail detection
- The article notes 'While the proportions differed significantly (χ2, p < 2e−16), the discrimination was imperfect' — hallmark filtering improves precision but does not perfectly separate true from false positives
- Exact χ-squared implementation details and parameters for some comparisons are not given

## Evidence

- [results] Filter performance finding: [evidence span withheld — anchor status: not_found]
- [results] Statistical validation: [evidence span withheld — anchor status: not_found]
