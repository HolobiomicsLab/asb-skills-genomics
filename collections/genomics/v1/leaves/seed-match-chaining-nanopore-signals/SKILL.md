---
name: seed-match-chaining-nanopore-signals
description: Use when mapping raw nanopore signals (FAST5/POD5/SLOW5/BLOW5) to a reference
  and you need to chain seed-match anchors into candidate mappings, or when verifying
  the f(i) chain-score DP recurrence (Section 2.2, Equation 3) against a manual trace.
when_to_use_negative:
- The input is basecalled nucleotide sequences rather than raw nanopore signals —
  RawHash/RawHash2 chaining operates over signal-level seed matches, not FASTQ/FASTA
  alignments
- You need base-level alignment refinement rather than seed-chain collation — the
  DTW signal alignment step (RawAlign-style) handles that stage, not chaining
- No reference genome is available and you need all-vs-all signal overlaps — use Rawsamble's
  overlapping mechanism instead of reference-guided chaining
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Performs the chaining over seed matches during signal-to-reference mapping;
    its DP recurrence f(i) is the algorithm reconstructed by this skill
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Predecessor hash-based signal mapping mechanism whose chaining and PAF conventions
    RawHash2 improves upon
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Minimap2
  role: Source of the minimap2-style DP and RMQ-based chaining formulation and code
    snippets reused by RawHash2
  repo: https://github.com/lh3/minimap2
- name: UNCALLED
  role: Defines the PAF template RawHash2 follows when reporting chaining-based mapping
    results
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:46644c33948340444cc619612756207ebcdb9bd73339564064ce0b67d54c7a21
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# seed-match-chaining-nanopore-signals

## Summary

Reconstructs and applies RawHash2's minimap2-style dynamic-programming chaining of seed-match anchors from raw nanopore signal mapping, computing per-anchor chain scores f(i) with gap-distance-based penalty terms. This is the collation step that turns hash-based seed matches into a mapping entry reported in UNCALLED-style PAF output.

## When to use

Use when mapping raw nanopore signals (FAST5/POD5/SLOW5/BLOW5) to a reference and you need to chain seed-match anchors into candidate mappings, or when verifying the f(i) chain-score DP recurrence (Section 2.2, Equation 3) against a manual trace.

## When NOT to use

- The input is basecalled nucleotide sequences rather than raw nanopore signals — RawHash/RawHash2 chaining operates over signal-level seed matches, not FASTQ/FASTA alignments
- You need base-level alignment refinement rather than seed-chain collation — the DTW signal alignment step (RawAlign-style) handles that stage, not chaining
- No reference genome is available and you need all-vs-all signal overlaps — use Rawsamble's overlapping mechanism instead of reference-guided chaining

## Inputs

- Seed-match anchors as (x, y, w) triples (reference signal position, query signal position, anchor weight)
- Reference genome index file (ref.ind) built from ref.fasta with an ONT pore model (e.g., r9.4 180mV 450bps 6-mer template_median68pA.model)
- Raw nanopore signal files in FAST5 (or POD5/SLOW5/BLOW5) format, e.g., test/data/d1_sars-cov-2_r94/fast5_files

## Outputs

- Chain scores f(i) for all anchors (chain_scores.tsv)
- Backtracked best chains over seed matches
- Mapping entries in UNCALLED-style PAF format (mapping.paf)

## How to apply

First build a reference index (e.g., ref.ind) with rawhash2 using the legacy R9.4 180mV 450bps 6-mer pore model (template_median68pA.model) and multiple threads (e.g., 32), mirroring the documented indexing command, then run rawhash2 mapping on raw signal files (e.g., test/data/d1_sars-cov-2_r94/fast5_files) to obtain seed matches. Extract the seed-match anchors as (x, y, w) triples, where x and y are reference and query signal positions and w is the anchor weight. Implement the DP recurrence f(i)=max{max_{i>j>=1}{f(j)+alpha(j,i)-beta(j,i)}, w_i}, where the penalty beta(j,i) is based on the gap distance between anchors i and j, adopting the minimap2-style DP and RMQ-based chaining formulation that RawHash2 reuses. Finally, backtrack the best chains, verify f(i) values on a small hand-checkable anchor set against a manual trace, and emit chain scores/backtracked chains while confirming the chain reproduces a valid entry in the generated mapping.paf in the UNCALLED-style PAF template.

## Related tools

- **RawHash2** (Performs the chaining over seed matches during signal-to-reference mapping; its DP recurrence f(i) is the algorithm reconstructed by this skill) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Predecessor hash-based signal mapping mechanism whose chaining and PAF conventions RawHash2 improves upon) — https://github.com/CMU-SAFARI/RawHash
- **Minimap2** (Source of the minimap2-style DP and RMQ-based chaining formulation and code snippets reused by RawHash2) — https://github.com/lh3/minimap2
- **UNCALLED** (Defines the PAF template RawHash2 follows when reporting chaining-based mapping results) — https://github.com/skovaka/UNCALLED

## Examples

```
./sigmap -i -r ref.fasta -p extern/kmer_models/r9.4_180mv_450bps_6mer/template_median68pA.model -o ref_index
```

## Evaluation signals

- Computed f(i) values on a small hand-checkable anchor set must exactly match a manual evaluation of the DP recurrence f(i)=max{max_{i>j>=1}{f(j)+alpha(j,i)-beta(j,i)}, w_i} (no mismatched f(i) values)
- The chained seeds must reproduce a valid mapping entry in the generated PAF-format output (mapping.paf in the UNCALLED-style template)
- Backtracked best chains must be consistent with the argmax predecessors recorded during the DP forward pass
- Gap penalty beta(j,i) must be nonzero and vary with the gap distance between anchors i and j

## Limitations

- Chaining operates on seed matches between the raw signal and the reference; sensitivity is bounded by the seeding mechanism and frequency filters used to reduce ambiguous seed hits
- The PAF output follows the UNCALLED-style template and reports approximate mapping information, not base-level alignment
- Minimizer-based seeding mode is faster (~3X) but slightly less accurate than the non-minimizer mode, affecting the anchors available for chaining

## Evidence

- [methods] To map the raw signal to a reference genome, RawHash performs chaining over the seed matches.: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] PAF template follows UNCALLED: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] RawHash overview: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] chaining code reuses minimap2 snippets: "some code snippets from [Minimap2](https://github.com/lh3/minimap2)"
