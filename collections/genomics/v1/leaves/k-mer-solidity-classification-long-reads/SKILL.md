---
name: k-mer-solidity-classification-long-reads
description: Use when polishing MSA-derived consensus sequences of long-read windows
  and remaining sequencing errors must be located as weak (non-solid) k-mer regions
  bounded by solid-k-mer anchors.
when_to_use_negative:
- Reads or contigs have not been decomposed into windows with MSA consensus sequences
  — solidity classification operates per window consensus, not directly on raw reads
- The goal is initial overlap/consensus computation rather than polishing of remaining
  errors after consensus
- Windows contain no weak regions (all k-mers meet the solid threshold) — graph exploration
  is unnecessary
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: 'Implements the polishing phase: k-mer solidity classification (--solid),
    local de Bruijn graph construction per window (merSize), and backtracking path
    search between solid anchors'
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Produces the read overlaps/alignment piles that CONSENT divides into windows
    whose consensus sequences are then polished
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:7b204b7d33692780f4a31cd12db3857a38eab0c5f1cb52de7f8299516f4ba795
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# k-mer-solidity-classification-long-reads

## Summary

Classify k-mers in a window consensus as solid when they occur at least a threshold number of times (CONSENT default --solid 4), then use consecutive solid k-mers as anchors to delimit and repair weak-error regions via a small-k local de Bruijn graph (merSize default 9).

## When to use

Use when polishing MSA-derived consensus sequences of long-read windows and remaining sequencing errors must be located as weak (non-solid) k-mer regions bounded by solid-k-mer anchors.

## When NOT to use

- Reads or contigs have not been decomposed into windows with MSA consensus sequences — solidity classification operates per window consensus, not directly on raw reads
- The goal is initial overlap/consensus computation rather than polishing of remaining errors after consensus
- Windows contain no weak regions (all k-mers meet the solid threshold) — graph exploration is unnecessary

## Inputs

- Consensus sequences of individual MSA windows from overlapping long reads (PacBio or ONT)
- All sequences of a single window used to build the local de Bruijn graph
- CONSENT polishing parameters: --merSize (k-mer size, default 9) and --solid (minimum occurrences for solidity, default 4)

## Outputs

- Polished window consensus sequences with weak k-mer regions replaced by graph-recovered paths
- Unchanged consensus sequences for regions whose flanking solid anchors could not be linked

## How to apply

Within each window, extract k-mers of size merSize (CONSENT default 9, used for both chaining and polishing) and count their occurrences across the window's sequences. Mark a k-mer as solid when its occurrence count meets the --solid threshold (default 4). Detect weak regions by scanning for stretches bounded on each side by n=3 consecutive solid k-mer anchors. Build a local de Bruijn graph from all sequences of the window and search it with backtracking for a path linking the two flanking anchor k-mers of each weak region; substitute the recovered path sequence into the consensus. If no connecting path exists, leave the weak region unchanged (unpolished) so errors are never 'corrected' with unsupported sequence.

## Related tools

- **CONSENT** (Implements the polishing phase: k-mer solidity classification (--solid), local de Bruijn graph construction per window (merSize), and backtracking path search between solid anchors) — https://github.com/morispi/CONSENT
- **minimap2** (Produces the read overlaps/alignment piles that CONSENT divides into windows whose consensus sequences are then polished) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB --merSize 9 --solid 4
```

## Evaluation signals

- Every substituted weak region is flanked by at least n=3 consecutive solid k-mers on both sides, with k-mer size equal to merSize (default 9)
- Solid k-mers have occurrence counts >= --solid (default 4) across the window's sequences
- Regions where backtracking finds no anchor-to-anchor path remain byte-identical to the original consensus (no invented sequence)
- Output is a valid FASTA of polished consensus/contigs, and CONSENT-polish completes with bounded memory (README example: ~150 MB, 4 cores on the 10x simulated PacBio example)
- End-to-end polishing reduces the final error rate of corrected reads or polished assemblies relative to the pre-polishing consensus

## Limitations

- Runtime and quality depend strongly on window size and minimum anchors parameters; the optimal runtime/quality compromise was not established in the paper
- Regions with no graph path connecting their flanking solid anchors are left unpolished, so errors in them persist
- The 5 TB disk requirement for a full human dataset (driven by the overlaps file) applies to the upstream correction workflow feeding the polishing step

## Evidence

- [methods] solid threshold: "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [methods] merSize parameter: "--merSize INT, -k INT:         k-mer size for chaining and polishing. (default: 9)"
- [intro] local de Bruijn graph polishing: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [discussion] polishing at window scale: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] README pipeline description: "First, a multiple alignment strategy is used in order to compute consensus. Then, this consensus is further polished with a local de Bruijn graph, in order to get rid of the remaining errors."
