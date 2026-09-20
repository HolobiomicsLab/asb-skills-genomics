---
name: anchor-flanked-weak-region-detection
description: Use when polishing MSA-derived window consensus sequences from long-read
  self-correction or assembly polishing, and remaining sequencing errors must be removed
  after consensus computation.
when_to_use_negative:
- The input is raw long reads rather than MSA-derived window consensus sequences —
  correction of raw reads starts from overlap computation and alignment piles, not
  polishing
- The window contains no weak-k-mer regions or too few anchors to permit consensus
  computation (minAnchors default 2), so there is nothing for the local graph to repair
- The goal is short-read-based correction via alignment of short reads to long reads
  (e.g. CoLoRMAP/HECiL-style approaches), which does not use self-correction local
  de Bruijn graph polishing
tools:
- name: CONSENT
  role: 'Implements the polishing phase: builds the per-window local de Bruijn graph,
    classifies solid k-mers, detects anchor-flanked weak regions, and replaces them
    via backtracking path recovery'
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:ab6d80f70bcb28144049d6d8585cd2830ce47298928861c4f16d8bcb94dcbbfb
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
yes_rate: 1.0
n_claims: 8
---

# anchor-flanked-weak-region-detection

## Summary

Detects low-quality (weak) k-mer regions in CONSENT window consensus sequences by classifying k-mers as solid based on occurrence count, and repairs them by replacing each region with a path recovered from a local de Bruijn graph between its flanking solid-k-mer anchors.

## When to use

Use when polishing MSA-derived window consensus sequences from long-read self-correction or assembly polishing, and remaining sequencing errors must be removed after consensus computation.

## When NOT to use

- The input is raw long reads rather than MSA-derived window consensus sequences — correction of raw reads starts from overlap computation and alignment piles, not polishing
- The window contains no weak-k-mer regions or too few anchors to permit consensus computation (minAnchors default 2), so there is nothing for the local graph to repair
- The goal is short-read-based correction via alignment of short reads to long reads (e.g. CoLoRMAP/HECiL-style approaches), which does not use self-correction local de Bruijn graph polishing

## Inputs

- Windowed consensus sequences from multiple sequence alignments of overlapping long reads (per-window, e.g. FASTA-derived)
- All sequences of a single window used to build the local de Bruijn graph
- CONSENT polishing parameters: merSize (k-mer size for chaining and polishing, default 9), solid (minimum k-mer occurrences, default 4)

## Outputs

- Polished window consensus sequence with weak k-mer regions replaced by recovered de Bruijn graph paths
- Unchanged window consensus for weak regions whose flanking anchors have no connecting path

## How to apply

Build a small-k local de Bruijn graph from all sequences of a single window, using the k-mer size for chaining and polishing (merSize default 9). Classify k-mers as solid when they occur at least 'solid' times (default 4), then detect weak k-mer regions bounded on each side by n=3 consecutive solid k-mer anchors. Search the local graph with a backtracking strategy for a path connecting the two flanking anchor k-mers of each weak region, and substitute the weak region with the recovered path sequence. When no connecting path links the anchors of a region, leave that region unpolished (unchanged) rather than forcing a low-confidence replacement. This step follows MSA consensus computation per window and precedes local realignment of the polished consensus back onto the read.

## Related tools

- **CONSENT** (Implements the polishing phase: builds the per-window local de Bruijn graph, classifies solid k-mers, detects anchor-flanked weak regions, and replaces them via backtracking path recovery) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta
```

## Evaluation signals

- Weak regions are substituted only when their two flanking solid k-mer anchors are connected by a path in the local de Bruijn graph
- Weak regions with no connecting path between their anchors are output unchanged (original sequence preserved)
- K-mers classified as solid meet or exceed the --solid occurrence threshold (default 4) computed over the window sequences
- The local de Bruijn graph is built with the polishing k-mer size (merSize default 9), and weak regions are bounded by n=3 consecutive solid k-mer anchors
- On small synthetic window inputs, verification confirms no sequence change outside detected weak regions

## Limitations

- The runtime of CONSENT's correction step tends to be higher than that of other state-of-the-art methods
- Runtime and result quality depend on window and anchor parameters; a satisfying compromise between runtime and quality must be sought empirically
- If no path links a weak region's flanking anchors, the region cannot be polished and errors there persist

## Evidence

- [methods] solid k-mer threshold: "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [intro] polishing with local de Bruijn graphs: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [discussion] per-window local de Bruijn graph polishing: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] README polishing description: "Then, this consensus is further polished with a local de Bruijn graph, in order to get rid of the remaining errors."
