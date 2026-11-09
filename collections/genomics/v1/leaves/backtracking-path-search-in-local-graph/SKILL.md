---
name: backtracking-path-search-in-local-graph
description: Use when a window consensus from a long-read self-correction pipeline
  still contains weak-k-mer error regions that must be replaced by paths recovered
  from a local de Bruijn graph bounded by solid-k-mer anchors.
when_to_use_negative:
- The region's flanking anchor k-mers have no connecting path in the local de Bruijn
  graph — the skill's rule is to leave the region unpolished rather than force a substitution
- The input needs no polishing (e.g. sequences already passed through CONSENT's consensus
  step with no remaining weak regions)
- Correction must come from external short-read k-mers (e.g. LoRDEC/FMLRC-style hybrid
  correction) — this skill is self-correction from the long reads' own window sequences
tools:
- name: CONSENT
  role: 'Implements the polishing phase: builds the per-window local de Bruijn graph,
    marks solid k-mers, and backtracks paths between anchors to replace weak regions'
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:07c1b4ab2259bf82a8c27e06dd33998cc90bdd85ea94d12081623b8b0b2fdaea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# backtracking-path-search-in-local-graph

## Summary

Polishes per-window consensus sequences produced by CONSENT's multiple-sequence-alignment step using a small-k local de Bruijn graph, in which solid k-mers anchor the graph and a backtracking search reconnects low-quality, weak-k-mer regions.

## When to use

Use when a window consensus from a long-read self-correction pipeline still contains weak-k-mer error regions that must be replaced by paths recovered from a local de Bruijn graph bounded by solid-k-mer anchors.

## When NOT to use

- The region's flanking anchor k-mers have no connecting path in the local de Bruijn graph — the skill's rule is to leave the region unpolished rather than force a substitution
- The input needs no polishing (e.g. sequences already passed through CONSENT's consensus step with no remaining weak regions)
- Correction must come from external short-read k-mers (e.g. LoRDEC/FMLRC-style hybrid correction) — this skill is self-correction from the long reads' own window sequences

## Inputs

- Per-window consensus sequence (or MSA-derived window sequences) from CONSENT's first correction phase
- Long reads (FASTA or FASTQ) to be corrected, or contigs (FASTA/FASTQ) plus long reads for assembly polishing
- Polishing parameters: merSize (default 9), solid (default 4)

## Outputs

- Polished consensus sequences with weak-k-mer regions replaced by backtracked graph paths
- Corrected long reads (result FASTA) or polished contigs (result FASTA)

## How to apply

Build a small-k local de Bruijn graph from all sequences of one window using the k-mer size for chaining and polishing (merSize default 9). Classify k-mers as solid with the minimum occurrence threshold during polishing (solid default 4), and detect weak-k-mer regions bounded on each side by n=3 consecutive solid k-mer anchors. For each weak region, run a backtracking search over the local graph to find a path connecting the two flanking anchor k-mers. Substitute the recovered path sequence into the weak region; if no connecting path exists between the anchors, leave the region unpolished (unchanged). The rationale is that solid k-mers are frequent enough to be trusted, so they pin the search while the graph supplies corrected sequence for the low-quality stretch in between.

## Related tools

- **CONSENT** (Implements the polishing phase: builds the per-window local de Bruijn graph, marks solid k-mers, and backtracks paths between anchors to replace weak regions) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Each weak-k-mer region is substituted only when its flanking solid anchors are linked by a graph path; regions without a linking path appear unchanged in the output
- Solid k-mer classification matches the threshold (solid=4 occurrences) and graph construction uses merSize=9
- Replaced regions contain only sequences traceable to paths between the two flanking anchors in the local graph
- Final corrected reads show reduced error rate relative to the pre-polishing consensus while preserving regions that had no connecting path
- Run on small synthetic window inputs reproduces the expected substitution/no-substitution behavior deterministically

## Limitations

- A weak region bounded by anchors with no connecting path in the local graph remains unpolished (original sequence kept)
- CONSENT's correction step runtime tends to be higher than that of other state-of-the-art methods
- Window size and minimum number of anchors parameters significantly impact runtime, and an optimal runtime/quality compromise was not established
- Selecting only the N highest identity overlaps may prove inefficient when repeated regions grow longer

## Evidence

- [intro] polish_consensus_local_dbg_intro: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [methods] solid_threshold_methods: "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [methods] mersize_methods: "--merSize INT, -k INT:         k-mer size for chaining and polishing. (default: 9)"
- [readme] polish_readme: "Then, this consensus is further polished with a local de Bruijn graph, in order to get rid of the remaining errors."
