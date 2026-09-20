---
name: de-bruijn-graph-construction-from-window-sequences
description: Use when polishing MSA-derived consensus sequences (or contigs) that
  still contain errors, and a local de Bruijn graph over a single window's sequences
  can recover low-quality regions via solid k-mer anchoring.
when_to_use_negative:
- The input is raw long reads requiring the first correction phase (overlap computation
  and MSA consensus) rather than polishing of already-computed consensuses.
- A weak region has no connecting path between its flanking solid anchors — the region
  is left unpolished, so the graph search cannot help.
- Window contains fewer than the required number of solid anchors (--minAnchors, default
  2), so consensus/polishing cannot proceed meaningfully.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: CONSENT
  role: Implements the local de Bruijn graph polishing of window consensus sequences
    with parameters merSize (k-mer size, default 9) and solid (solidity threshold,
    default 4), via CONSENT-correct or CONSENT-polish.
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:4822120a848c9a75a9ef14c5c793ee1bce62ee3fa816ad615e72088a9e714d8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# de-bruijn-graph-construction-from-window-sequences

## Summary

Builds a small-k local de Bruijn graph from the sequences of a single MSA window and polishes the consensus by replacing weak-k-mer regions with paths anchored on solid k-mers, as done in CONSENT's second correction phase.

## When to use

Use when polishing MSA-derived consensus sequences (or contigs) that still contain errors, and a local de Bruijn graph over a single window's sequences can recover low-quality regions via solid k-mer anchoring.

## When NOT to use

- The input is raw long reads requiring the first correction phase (overlap computation and MSA consensus) rather than polishing of already-computed consensuses.
- A weak region has no connecting path between its flanking solid anchors — the region is left unpolished, so the graph search cannot help.
- Window contains fewer than the required number of solid anchors (--minAnchors, default 2), so consensus/polishing cannot proceed meaningfully.

## Inputs

- Per-window consensus sequences from multiple sequence alignment of overlapping long reads (FASTA)
- All sequence strings of a single window (reads contributing to the pile, or contig plus aligning reads for polishing)

## Outputs

- Polished window consensus sequence with weak-k-mer regions replaced by graph paths
- Final corrected long reads (FASTA) or polished contigs after local realignment of polished consensuses

## How to apply

For each window (default size 500 bp, 50 bp overlap), build a local de Bruijn graph from all window sequences using the polishing k-mer size (--merSize, default 9). Classify k-mers as solid when they occur at least --solid (default 4) times, and detect weak-k-mer regions bounded on each side by n=3 consecutive solid k-mer anchors. Search the graph with a backtracking strategy for a path connecting the two flanking anchors of each weak region; if a path is found, replace the weak region with the recovered sequence, otherwise leave the region unchanged. The polished consensus is then locally realigned back to the read (or contig) for final correction.

## Related tools

- **CONSENT** (Implements the local de Bruijn graph polishing of window consensus sequences with parameters merSize (k-mer size, default 9) and solid (solidity threshold, default 4), via CONSENT-correct or CONSENT-polish.) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB --merSize 9 --solid 4
```

## Evaluation signals

- Weak regions are substituted only when their flanking solid-k-mer anchors are linked by a path in the local graph; regions without a connecting path are output unchanged.
- Solid k-mer classification matches the --solid threshold (default 4 occurrences) and graph vertices use --merSize k (default 9).
- Polished consensus sequences can be locally realigned to the original read without large indels introduced by the polishing step.
- On synthetic window inputs, expected output equals input sequence wherever no weak region is detected or no anchor-linking path exists.
- Downstream error rate of corrected reads or polished Flye assemblies decreases relative to unpolished consensuses.

## Limitations

- The strategy of selecting only the N highest-identity overlaps can be inefficient for long repeated regions, reducing the coverage available for a window.
- Window size and minimum number of anchors significantly impact runtime; an optimal compromise between runtime and result quality has not been established.
- CONSENT's correction-step runtime tends to be higher than other state-of-the-art methods, and self-correction of a full human dataset requires ~5 TB of disk space, driven by the overlaps file.

## Evidence

- [intro] These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate.: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [methods] --solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4): "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [discussion] it is further polished with the help of a local de Bruijn graph, at the scale of the window: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] --merSize INT, -k INT:         k-mer size for chaining and polishing. (default: 9): "--merSize INT, -k INT:         k-mer size for chaining and polishing. (default: 9)"
- [full_text] a local de Bruijn graph (per window) is explored using solid k-mers — k-mers occurring more frequently than a given threshold — as anchor points, in order to correct low-quality, weak k-mer regions.: [evidence span withheld — anchor status: not_found]
