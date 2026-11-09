---
name: local-de-bruijn-graph-polishing
description: Use when windowed MSA consensus sequences still carry residual substitution/indel
  errors, or when a raw long-read assembly (e.g., Flye output) must be polished with
  ONT or PacBio reads.
when_to_use_negative:
- Input is already a corrected/polished read set or high-accuracy reads where residual
  error rate is negligible
- Task requires correcting reads rather than assemblies and no overlapping long-read
  support is available to build the graph
edam_operation: http://edamontology.org/operation_3185
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Implements CONSENT-correct and CONSENT-polish, which polish MSA-derived consensus
    sequences or raw contigs with local de Bruijn graphs
  repo: https://github.com/morispi/CONSENT
- name: Flye
  role: Source of raw assemblies whose quality is improved by CONSENT's de Bruijn
    graph polishing
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:ba6732353f7a6f6ceed4483a1ad7377960e38890690653416f9bd8d8601d80de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - ONT
  edam_operation: http://edamontology.org/operation_3185
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# local-de-bruijn-graph-polishing

## Summary

Polish consensus sequences with local de Bruijn graphs, as implemented in CONSENT, to remove remaining errors after multiple sequence alignment-based correction of long reads or polishing of raw assemblies.

## When to use

Use when windowed MSA consensus sequences still carry residual substitution/indel errors, or when a raw long-read assembly (e.g., Flye output) must be polished with ONT or PacBio reads.

## When NOT to use

- Input is already a corrected/polished read set or high-accuracy reads where residual error rate is negligible
- Task requires correcting reads rather than assemblies and no overlapping long-read support is available to build the graph

## Inputs

- Window consensus sequences from MSA of overlapping long reads (self-correction mode)
- Raw assembly FASTA contigs (polishing mode, --contigs)
- Long reads in FASTA/FASTQ (PacBio or ONT) used to build the local de Bruijn graph

## Outputs

- correctedReads.fasta with polished consensus per read
- polishedAssembly.fasta with corrected contigs

## How to apply

In CONSENT, each window consensus derived from the multiple sequence alignment of an alignment pile is further polished with a local de Bruijn graph built at the scale of the window, to correct remaining errors and reduce the final error rate. A k-mer is treated as solid when it occurs at least --solid (default 4) times, and the k-mer size for both chaining and polishing is --merSize (default 9). After graph polishing, the polished consensus is locally realigned to the original read to emit the corrected read (CONSENT-correct --type PB/ONT) or, for polishing, contigs are corrected with CONSENT-polish. The same mechanism improves Flye assemblies built from raw reads, and is 2-38x faster than other polishing tools while providing comparable results.

## Related tools

- **CONSENT** (Implements CONSENT-correct and CONSENT-polish, which polish MSA-derived consensus sequences or raw contigs with local de Bruijn graphs) — https://github.com/morispi/CONSENT
- **Flye** (Source of raw assemblies whose quality is improved by CONSENT's de Bruijn graph polishing)

## Examples

```
./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta
```

## Evaluation signals

- Polishing step completes and the --solid threshold (default 4) k-mer graph is built without missing k-mer size parameter (default --merSize 9)
- On the README example, CONSENT-polish finishes in ~15 sec using at most 150 MB of RAM on 4 cores
- Post-correction error rate measured with ELECTER decreases relative to the pre-polish consensus (e.g., 1.77x reduction with segmentation enabled)
- Polished Flye assemblies show improved quality metrics compared to raw assemblies
- Polished consensus successfully locally realigns to the original read/contig before writing output

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods
- Runtime and memory grow with dataset size; a full human ONT dataset (reads up to 1.5 Mbp) needs ~10 days, <100 GB RAM, and ~5 TB disk for the overlaps file
- Polishing quality depends on window and anchor parameters (windowSize 500, minSupport 4, anchorSupport 8, minAnchors 2); the optimal runtime/quality compromise is not established

## Evidence

- [intro] These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate.: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [methods] Solid k-mer threshold parameter: "--solid INT, -f INT:           Minimum number of occurrences to consider a k-mer as solid during polishing. (default: 4)"
- [readme] Polishing usage from README: "./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta"
- [abstract] Polishing speed finding: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [discussion] Polished consensus realignment: "Finally, the polished consensus is locally realigned to the read, in order to correct it."
