---
name: synthetic-sequence-polishing-validation
description: 'Use when testing a reconstructed CONSENT-style polishing module: you
  need to confirm that weak-k-mer regions between solid anchors are corrected via
  local de Bruijn graph paths, and left unchanged otherwise.'
when_to_use_negative:
- Input is a full long-read correction job rather than a single-window polishing validation
  — use the complete CONSENT-correct pipeline instead
- Sequences contain no detectable weak k-mer regions, so there is nothing to polish
  and the validation invariant cannot be exercised
- You need assembly polishing of contigs against reads — use CONSENT-polish on contig/read
  FASTA files rather than this synthetic validation setup
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: CONSENT
  role: Reference implementation whose second correction phase (local de Bruijn graph
    polishing of MSA consensus sequences) is being reconstructed and validated
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:ab66cae23fb06d04bc9f8840a897232652e4ee72f27d59b014963ff0c83e07d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# synthetic-sequence-polishing-validation

## Summary

Validates a reimplementation of CONSENT's consensus-polishing phase by running the local de Bruijn graph polishing on small synthetic window sequences and checking that weak k-mer regions are replaced only when a solid-anchor-flanked path exists.

## When to use

Use when testing a reconstructed CONSENT-style polishing module: you need to confirm that weak-k-mer regions between solid anchors are corrected via local de Bruijn graph paths, and left unchanged otherwise.

## When NOT to use

- Input is a full long-read correction job rather than a single-window polishing validation — use the complete CONSENT-correct pipeline instead
- Sequences contain no detectable weak k-mer regions, so there is nothing to polish and the validation invariant cannot be exercised
- You need assembly polishing of contigs against reads — use CONSENT-polish on contig/read FASTA files rather than this synthetic validation setup

## Inputs

- Small synthetic window sequences (e.g. FASTA) embedding weak k-mer regions flanked by solid k-mer anchors
- Polishing parameters: merSize (default 9), solid threshold (default 4), anchor flanking count n = 3

## Outputs

- Polished window consensus sequences with weak regions substituted by graph-recovered paths
- Unchanged sequences for weak regions whose anchors cannot be linked by a graph path

## How to apply

Build a small-k local de Bruijn graph (k = merSize, default 9) from all sequences of a single window, and classify k-mers as solid using the minimum occurrence threshold (--solid, default 4). Detect weak-k-mer regions bounded on each side by n = 3 consecutive solid k-mer anchors. Search the local graph with a backtracking strategy for a path connecting the two flanking anchors of each weak region, then replace each weak region with the recovered path sequence. If no connecting path links a region's anchors, the region must be left unpolished (unchanged) — this invariant is the core validation check. Generate small synthetic window inputs containing both correctable and uncorrectable weak regions so both outcomes are exercised.

## Related tools

- **CONSENT** (Reference implementation whose second correction phase (local de Bruijn graph polishing of MSA consensus sequences) is being reconstructed and validated) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB -k 9 -f 4
```

## Evaluation signals

- Weak regions flanked by 3 consecutive solid k-mers on each side are replaced with a path recovered from the local de Bruijn graph
- Weak regions whose flanking anchors cannot be linked are output unchanged (no spurious substitutions)
- K-mer solidity classification respects the --solid occurrence threshold (default 4) at k = merSize (default 9)
- Sequences without weak regions pass through the module unmodified
- Reconstructed module behavior matches reference CONSENT output on the same synthetic window inputs

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods
- The strategy of only selecting the N highest identity overlaps might prove inefficient for long repeated regions
- Optimal parameter compromise between window size, minimum number of anchors, and runtime is not established

## Evidence

- [methods] solid threshold: "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [methods] merSize: "--merSize INT, -k INT:         k-mer size for chaining and polishing. (default: 9)"
- [intro] polishing rationale: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [discussion] window-scale polishing: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] readme pipeline description: "Then, this consensus is further polished with a local de Bruijn
graph, in order to get rid of the remaining errors."
