---
name: chi-squared-true-false-positive-comparison
description: Use when a variant callset has been classified into true and false positives
  against a truth set (e.g., HGSVC 5,630-insertion set) and you must statistically
  justify using a sequence feature as a call filter.
when_to_use_negative:
- Calls have not been labeled against a truth set — there are no true/false positive
  classes to tabulate, so χ² comparison is meaningless
- The hallmark annotations themselves (polyA tails, TTTTT/AA motif) have not yet been
  extracted from reassembled sequences
- Evaluating a non-retrotransposition variant type (deletions, inversions) where polyA/L1-motif
  hallmarks do not apply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: RetroInspector
  role: Produces the reassembled insertion sequences and supporting-read counts that
    feed the TP/FP hallmark contingency tables
  repo: https://github.com/javiercguard/retroinspector
- name: RapidFuzz
  role: Fuzzy text search (initial 60% similarity ratio) for the L1 endonuclease motif
    TTTTT/AA and its reverse complement used to annotate calls before testing
- name: pROC
  role: Supports recomputation of precision/recall of the callset with and without
    the hallmark filters at 5 and 7 supporting reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:f5295358b3f606a5c2e84f924f6f6c022285bfd1ecd436186c1f04880f99e991
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chi-squared-true-false-positive-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a χ-squared test to compare the proportion of retrotransposition hallmarks (polyA tails, L1 endonuclease TTTTT/AA target motif) between true and false positives in a TE insertion callset, quantifying whether each hallmark discriminates real insertions from caller noise before adopting it as a filter.

## When to use

Use when a variant callset has been classified into true and false positives against a truth set (e.g., HGSVC 5,630-insertion set) and you must statistically justify using a sequence feature as a call filter.

## When NOT to use

- Calls have not been labeled against a truth set — there are no true/false positive classes to tabulate, so χ² comparison is meaningless
- The hallmark annotations themselves (polyA tails, TTTTT/AA motif) have not yet been extracted from reassembled sequences
- Evaluating a non-retrotransposition variant type (deletions, inversions) where polyA/L1-motif hallmarks do not apply

## Inputs

- Reassembled inserted sequences from the cuteSV-Sniffles2 merged insertion callset
- GRCh38 reference sequence ±20 bp around each insertion site
- HGSVC TE insertion truth set labels (true positive / false positive per call)
- Supporting-read counts per call at thresholds of 5 and 7 reads

## Outputs

- Per-hallmark 2x2 contingency table (feature present/absent vs true/false positive)
- χ-squared statistic and p-value per hallmark feature
- Precision and recall with and without hallmark filters at 5 and 7 supporting reads
- Filtered TE insertion callset retaining calls passing polyA-tail and L1-motif criteria

## How to apply

First classify each insertion call (from the cuteSV-Sniffles2 merged callset) as true or false positive by matching against the HGSVC truth set, then annotate each call for the hallmark features: polyA/polyT terminus runs (tails required to start within the last/first 10% of the reassembled inserted sequence, merging candidates separated by ≤2 bp gaps) and the L1 endonuclease consensus motif TTTTT/AA (and reverse complement TT/AAAAA) found by RapidFuzz fuzzy search at a 60% initial similarity ratio, keeping the hit closest to the insertion coordinates. Build a 2x2 contingency table per hallmark (feature present/absent vs TP/FP) and run a χ-squared test on each; the paper reports significant proportion differences (χ², p < 2e-16) yet imperfect discrimination. Use the per-hallmark significance alongside recomputed precision/recall at 5 and 7 supporting reads to decide which hallmarks to apply as filters, where combining them raised precision to 90.38% (5 reads) and 90.94% (7 reads). Report any recall loss incurred by the filtering as part of the judgment.

## Related tools

- **RetroInspector** (Produces the reassembled insertion sequences and supporting-read counts that feed the TP/FP hallmark contingency tables) — https://github.com/javiercguard/retroinspector
- **RapidFuzz** (Fuzzy text search (initial 60% similarity ratio) for the L1 endonuclease motif TTTTT/AA and its reverse complement used to annotate calls before testing)
- **pROC** (Supports recomputation of precision/recall of the callset with and without the hallmark filters at 5 and 7 supporting reads)

## Examples

```
chisq.test(matrix(c(tp.with, tp.without, fp.with, fp.without), nrow=2, byrow=TRUE))
```

## Evaluation signals

- Each hallmark yields a significant χ² result between TPs and FPs (the paper reports χ², p < 2e-16)
- After applying hallmark filters, precision rises above 90% at both 5 (90.38%) and 7 (90.94%) supporting reads
- PolyA tails are only accepted when they start within the last (or first) 10% of the inserted sequence, with ≤2 bp gaps merged — verify this constraint held
- The L1 motif match kept is the one closest to the insertion coordinates when multiple RapidFuzz hits exist
- Any recall loss from filtering is explicitly reported alongside the precision gain

## Limitations

- Significant χ² differences do not guarantee per-call discrimination: the authors note 'the discrimination was imperfect'
- HGSVC truth-set data was basecalled with an older basecaller, so homopolymer-rich features like polyA tails may be imperfect even after reassembly
- Recall drops when the hallmark filters are applied, so the test justifies a precision/recall tradeoff, not a strictly better callset

## Evidence

- [results] While the proportions differed significantly (χ 2, p < 2e−16), the discrimination was imperfect.: [evidence span withheld — anchor status: not_found]
- [results] we investigated two key retrotransposition features: polyA tails at the inserted sequence’s terminus, and the presence of the consensus motif for L1’s endonuclease target sequence.: "we investigated two key retrotransposition features: polyA tails at the inserted sequence’s terminus, and the presence of the consensus motif for L1’s endonuclease target sequence."
- [abstract] The user can also set the threshold for the number of supporting reads for the variant filtering.: "The user can also set the threshold for the number of supporting reads for the variant filtering."
