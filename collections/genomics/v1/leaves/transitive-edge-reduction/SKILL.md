---
name: transitive-edge-reduction
description: Use when building a string graph from long-read overlaps (minimap2 PAF
  overlaps of Nanopore R9/R10 or PacBio CLR reads) and full pairwise identity computation
  is too slow, or when removed edges have broken assembly paths.
when_to_use_negative:
- Reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves only
  mediocre performance on these and other strategies may be preferable
- Overlaps are already fully aligned with base-level identities and the graph is small
  enough that restricting local alignment to active edges offers no speedup
- Heterozygosity rate is very low, where PECAT's error correction may not distinguish
  small sequencing errors from heterozygote alleles
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Detects candidate read-to-read overlaps among corrected reads (e.g. -x ava-pb
    for PacBio CLR) that are then extended and filtered before string graph construction
  repo: https://github.com/lh3/minimap2
- name: PECAT
  role: 'Implements the string graph construction: transitive-edge marking via Myers''
    algorithm, adaptive identity threshold removal, contained-read bridging, and two-round
    reassembly'
  repo: https://github.com/lemene/PECAT
- name: NECAT
  role: Predecessor pipeline from which PECAT's string-graph-based assembly scheme
    derives; used for baseline extraction of 80X raw reads
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:f648a1dc901f4317968e3fb53bb18bd6295b9028209a9cbc8fccb34bec8c620e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NECAT
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# transitive-edge-reduction

## Summary

Reduces a read-overlap graph to a sparse string graph by detecting and marking transitive overlaps with Myers' algorithm, so expensive local alignment (identity computation) is performed only on the small set of active edges, then reuses transitive edges to reconnect paths broken by removed low-quality edges.

## When to use

Use when building a string graph from long-read overlaps (minimap2 PAF overlaps of Nanopore R9/R10 or PacBio CLR reads) and full pairwise identity computation is too slow, or when removed edges have broken assembly paths.

## When NOT to use

- Reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves only mediocre performance on these and other strategies may be preferable
- Overlaps are already fully aligned with base-level identities and the graph is small enough that restricting local alignment to active edges offers no speedup
- Heterozygosity rate is very low, where PECAT's error correction may not distinguish small sequencing errors from heterozygote alleles

## Inputs

- Corrected long reads (FASTA/FASTQ) from Nanopore R9/R10 or PacBio CLR sequencing
- Candidate read-to-read overlaps from minimap2 (PAF) computed with align_rd2rd_options
- PECAT configuration file (cfgfile) with align_filter_options and asm1_assemble_options

## Outputs

- String graph with transitive edges marked inactive and low-identity edges removed
- Haplotype-collapsed primary contigs ($PROJECT/3-assemble/primary.fasta)
- Reconnected assembly paths after bridging contained reads and low-quality edge removal
- Round-one overlaps reused for the round-two diploid reassembly

## How to apply

First detect candidate read-to-read overlaps among corrected reads with minimap2 using the align_rd2rd_options parameters, extend each alignment to the read ends with the diff algorithm, and drop overlaps that still carry long overhangs (align_filter_options). Build the string graph by marking transitive overlaps with Myers' algorithm as inactive edges, and remove edges whose identity falls below an adaptive threshold — only active edges need local-alignment-based identity computation, which is what makes the graph construction fast. Contained reads are bridged and low-coverage/low-quality edges removed; when this breaks paths, reuse the marked transitive edges and contained-read overlaps to reconnect them. In PECAT's two-round scheme, round-one overlaps are reused in round two after only removing inconsistent overlaps (strict threshold: 8 different SNP alleles on corrected reads; loose: 6 on raw reads), rather than recomputing all overlaps.

## Related tools

- **minimap2** (Detects candidate read-to-read overlaps among corrected reads (e.g. -x ava-pb for PacBio CLR) that are then extended and filtered before string graph construction) — https://github.com/lh3/minimap2
- **PECAT** (Implements the string graph construction: transitive-edge marking via Myers' algorithm, adaptive identity threshold removal, contained-read bridging, and two-round reassembly) — https://github.com/lemene/PECAT
- **NECAT** (Predecessor pipeline from which PECAT's string-graph-based assembly scheme derives; used for baseline extraction of 80X raw reads)

## Examples

```
minimap2 -x ava-ont corrected_reads.fq corrected_reads.fq > overlaps.paf
```

## Evaluation signals

- Round-one output $PROJECT/3-assemble/primary.fasta exists and is valid FASTA
- Round-two contig files under $PROJECT/5-assemble are produced and parse as valid FASTA
- Local alignment (identity computation) is executed only on active (non-transitive, non-contained) edges, not on the full overlap set — checkable via runtime/profiling of the identity step
- Overlaps removed in round two are restricted to inconsistent ones (>=8 different SNP alleles on corrected reads, or >=6 on raw reads), so the count of retained overlaps approaches the round-one count
- Paths broken by low-quality-edge removal are reconnected via transitive edges, verified by contig count/length metrics relative to a run without reconnection

## Limitations

- Edges below the adaptive identity threshold are removed as low quality, which can break paths; reconnection via transitive edges is a heuristic and may not recover all breaks
- Inconsistent-overlap detection relies on heterozygous SNP calling, whose accuracy is still low in low heterozygosity rate regions
- High-quality phased assembly with PECAT requires higher data coverage; running time increases nonlinearly with data volume, motivating the active-edge-only alignment strategy

## Evidence

- [methods] Transitive edges detected with Myers' algorithm are marked inactive so identity computation is only needed on active edges: [evidence span withheld — anchor status: not_found]
- [discussion] Alignments extended to read ends and overhang-filtered before graph construction: "we extend the alignment to the ends of the reads with the diff36 algorithm and filter out overlaps still with long overhangs."
- [discussion] Adaptive identity threshold removes low-quality string graph edges: "The edges whose identities are less than the threshold are marked as low quality and removed."
- [discussion] Round-two assembly reuses round-one overlaps, removing only inconsistent ones: "reuse the overlaps built in the first round of assembly and just simplify the overlaps by removing inconsistent ones before reassembling the diploid genome"
- [readme] minimap2 README documents the overlap-detection use case for long reads: [evidence span withheld — anchor status: external_doc]
