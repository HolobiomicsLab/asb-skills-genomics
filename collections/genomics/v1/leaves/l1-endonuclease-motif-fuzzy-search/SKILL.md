---
name: l1-endonuclease-motif-fuzzy-search
description: Use when filtering a merged TE insertion callset (e.g., cuteSV-Sniffles2
  insertions) and you need to discriminate true retrotransposition insertions from
  false positives by searching reassembled inserted sequences for the L1 endonuclease
  target motif.
when_to_use_negative:
- Input variants are not TE insertions (e.g., deletions or other SV classes) — the
  L1 target-site motif and polyA tail hallmarks are specific to retrotransposition
- Insertions are already annotated from a curated truth set or TE database lookup,
  so sequence-hallmark filtering would add no discrimination
- Working with short-read callsets whose inserted sequences cannot be reassembled
  from spanning reads
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: RetroInspector
  role: Snakemake pipeline that reassembles the affected region from supporting reads
    and applies the hallmark-based variant filtering
  repo: https://github.com/javiercguard/retroinspector
- name: RapidFuzz
  role: Fuzzy string matching library used to search reassembled insertions for the
    L1 endonuclease consensus motif at a 60% similarity ratio
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:cc898913fa47288eaad5782ee92f37df38a1dc0ba64e59f047315584d021000a
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
  - pROC
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# l1-endonuclease-motif-fuzzy-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A retrotransposition-hallmark filter from RetroInspector that scans reassembled insertion sequences with RapidFuzz for the L1 endonuclease target consensus motif TTTTT/AA (and its reverse complement TT/AAAAA) on both strands, then uses motif presence together with polyA tail detection to re-filter TE insertion callsets and boost precision above 90%.

## When to use

Use when filtering a merged TE insertion callset (e.g., cuteSV-Sniffles2 insertions) and you need to discriminate true retrotransposition insertions from false positives by searching reassembled inserted sequences for the L1 endonuclease target motif.

## When NOT to use

- Input variants are not TE insertions (e.g., deletions or other SV classes) — the L1 target-site motif and polyA tail hallmarks are specific to retrotransposition
- Insertions are already annotated from a curated truth set or TE database lookup, so sequence-hallmark filtering would add no discrimination
- Working with short-read callsets whose inserted sequences cannot be reassembled from spanning reads

## Inputs

- Reassembled inserted sequences from the cuteSV-Sniffles2 merged insertion callset
- GRCh38 reference sequence ±20 bp around each insertion site
- Insertion coordinates and supporting-read counts per variant call

## Outputs

- Per-insertion annotations of L1 endonuclease motif hits (position, similarity ratio, strand)
- Boolean hallmark flags (motif present, polyA tail present) per insertion
- Filtered TE insertion callsets with recomputed precision and recall at 5- and 7-read thresholds

## How to apply

Retrieve each reassembled inserted sequence from the cuteSV-Sniffles2 merged insertion callset plus the GRCh38 reference sequence ±20 bp around each insertion site. Run a RapidFuzz fuzzy text search at an initial similarity ratio of 60% for the L1 endonuclease consensus motif TTTTT/AA and its reverse complement TT/AAAAA, searching both strands; when multiple matches are returned, keep the hit closest to the insertion coordinates. Pair the motif result with polyA/polyT tail detection (adenine/thymine runs merged across gaps ≤2 bp, tail required to start within the terminal 10% of the inserted sequence) to label insertions with retrotransposition hallmarks. Finally, recompute precision and recall of the callset with and without the hallmark filter at supporting-read thresholds of 5 and 7 reads, and test hallmark enrichment in true vs false positives with a χ-squared test. The paper reports this filter lifts precision to 90.38% and 90.94% at 5 and 7 supporting reads, respectively.

## Related tools

- **RetroInspector** (Snakemake pipeline that reassembles the affected region from supporting reads and applies the hallmark-based variant filtering) — https://github.com/javiercguard/retroinspector
- **RapidFuzz** (Fuzzy string matching library used to search reassembled insertions for the L1 endonuclease consensus motif at a 60% similarity ratio)

## Examples

```
from rapidfuzz import fuzz; hit = max(motif_positions, key=lambda p: (fuzz.ratio('TTTTTAA', window(p)), -abs(p-center))) if fuzz.ratio('TTTTTAA', window) >= 60 else None
```

## Evaluation signals

- Precision of the TE insertion callset rises above 90% (reported: 90.38% at 5 reads, 90.94% at 7 reads) after applying the hallmark filter
- χ-squared test shows a significant difference in hallmark frequency between true and false positives (paper: p < 2e−16)
- Every motif hit retains the match closest to the insertion coordinates when multiple fuzzy matches exist
- Recall loss after filtering is reported and acceptable relative to the precision gain
- Motif search covers both the TTTTT/AA consensus and its reverse complement TT/AAAAA on both strands at the initial 60% similarity ratio

## Limitations

- Discrimination by hallmark features is imperfect even when statistically significant — recall is reduced by the filter
- HGSVC truth-set data was basecalled with an older basecaller, so homopolymer-rich regions (relevant to TTTTT/AA and polyA runs) may be imperfect even after reassembly
- χ-squared test implementation details and some comparison parameters are not specified in the paper

## Evidence

- [results] χ-squared significance but imperfect discrimination: [evidence span withheld — anchor status: not_found]
- [methods] RapidFuzz fuzzy search method for the L1 motif: [evidence span withheld — anchor status: not_found]
- [results] reported precision after hallmark filtering: [evidence span withheld — anchor status: not_found]
