---
name: minimap2-style-chain-score-computation
description: 'Use when implementing or verifying the chaining stage of a raw-signal
  (FAST5/POD5/SLOW5) mapper: seed matches must be converted into scored chains with
  a gap penalty before emitting PAF mappings.'
when_to_use_negative:
- The raw signals have already been mapped and chained — DTW signal alignment (as
  in RawAlign) is the next refinement stage, not chaining.
- The task is all-vs-all overlap finding without a reference (Rawsamble) rather than
  mapping seeds against an indexed reference.
- Anchors are nucleotide minimizers from sequence reads (plain minimap2 use) rather
  than raw-signal seed matches from a hash-table query.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RawHash2
  role: Generates the seed matches (via hash-value query of ref.ind) over which the
    chaining DP is performed; source of the f(i) recurrence with gap penalty beta(j,i)
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Minimap2
  role: Reference implementation of the minimap2-style DP/RMQ chaining formulation
    that RawHash2 reuses, including code snippets incorporated into RawHash2
  repo: https://github.com/lh3/minimap2
- name: RawHash
  role: Baseline hash-based raw signal mapper whose chaining-over-seed-matches workflow
    this DP reconstruction mirrors
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Defines the PAF output template in which the chained-seed mapping result is
    reported
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:00be2efba66dc1e2436ce86ec29eba7a48f5a4ea8b019a65c19ff8e811007415
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - Minimap2
  - RawHash
  - RawHash2
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# minimap2-style-chain-score-computation

## Summary

Reconstructs RawHash2's chaining DP, in which each seed-match anchor receives a chain score f(i) = max{max_{i>j>=1}{f(j)+alpha(j,i)-beta(j,i)}, w_i} with a gap-distance-based penalty beta, following the minimap2-style chaining (RMQ-based DP) that RawHash2 reuses to map raw nanopore signals.

## When to use

Use when implementing or verifying the chaining stage of a raw-signal (FAST5/POD5/SLOW5) mapper: seed matches must be converted into scored chains with a gap penalty before emitting PAF mappings.

## When NOT to use

- The raw signals have already been mapped and chained — DTW signal alignment (as in RawAlign) is the next refinement stage, not chaining.
- The task is all-vs-all overlap finding without a reference (Rawsamble) rather than mapping seeds against an indexed reference.
- Anchors are nucleotide minimizers from sequence reads (plain minimap2 use) rather than raw-signal seed matches from a hash-table query.

## Inputs

- Seed-match anchors as (x, y, w) triples: reference signal position, query signal position, and anchor weight
- RawHash2 index file (ref.ind) generated from a reference genome (e.g. ref.fasta) with an R9.4 6-mer pore model
- Raw nanopore signal reads from FAST5 (or POD5/SLOW5) files queried against the index

## Outputs

- Chain scores f(i) and backtracked best chains for all anchors (e.g. chain_scores.tsv)
- Mapping entries in the UNCALLED-style PAF template (mapping.paf) derived from the chained seeds

## How to apply

Represent seed matches as (x, y, w) anchors, where x is the reference signal position, y is the query signal position, and w is the anchor weight, obtained after RawHash2's indexing (built with the R9.4 180mV 450bps 6-mer pore model, e.g. template_median68pA.model) and mapping steps. Implement the DP recurrence f(i) = max{max_{i>j>=1}{f(j)+alpha(j,i)-beta(j,i)}, w_i}, where beta(j,i) is the penalty derived from the gap distance between anchors i and j, adopting minimap2's DP-and-RMQ chaining formulation that RawHash2 reuses. Track backpointers so the best chain can be backtracked after scoring. Validate on a small hand-checkable anchor set by manually evaluating the recurrence and comparing every f(i). Confirm the chained seeds reproduce a valid mapping entry in the UNCALLED-style PAF template that RawHash2 outputs.

## Related tools

- **RawHash2** (Generates the seed matches (via hash-value query of ref.ind) over which the chaining DP is performed; source of the f(i) recurrence with gap penalty beta(j,i)) — https://github.com/CMU-SAFARI/RawHash
- **Minimap2** (Reference implementation of the minimap2-style DP/RMQ chaining formulation that RawHash2 reuses, including code snippets incorporated into RawHash2) — https://github.com/lh3/minimap2
- **RawHash** (Baseline hash-based raw signal mapper whose chaining-over-seed-matches workflow this DP reconstruction mirrors) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Defines the PAF output template in which the chained-seed mapping result is reported) — https://github.com/skovaka/UNCALLED

## Examples

```
./rawhash2 -t 32 ref.ind test/data/d1_sars-cov-2_r94/fast5_files -o mapping.paf
```

## Evaluation signals

- On a small hand-checkable anchor set, every computed f(i) matches a manual trace of f(i) = max{max_{i>j>=1}{f(j)+alpha(j,i)-beta(j,i)}, w_i} with no mismatched values.
- Backtracking from the anchor with the maximal f(i) yields a chain with monotonically increasing reference (x) and query (y) positions consistent with the gap penalty beta.
- The chained seeds produce a valid mapping entry in the UNCALLED-style PAF template (mapping.paf) after mapping against ref.ind.
- Anchor weights w in the (x, y, w) triples are consistent with the seed matches emitted by RawHash2 mapping of test/data/d1_sars-cov-2_r94/fast5_files.

## Limitations

- The recurrence and penalty scores come from the chaining stage only; downstream DTW-based signal alignment (integrated from RawAlign) is required for base-level alignment quality.
- Chaining operates on quantized raw-signal coordinates; RawHash2's adaptive quantization affects seed positions upstream, so errors there propagate into the anchors.
- RawHash2's minimizer seeding mode is slightly less accurate than the non-minimizer mode (though ~3X faster), which can change the anchor set feeding the DP.

## Evidence

- [methods] chaining over seed matches: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] Minimap2 code reused: "some code snippets from [Minimap2](https://github.com/lh3/minimap2)"
- [readme] mapping step queries hash table: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
