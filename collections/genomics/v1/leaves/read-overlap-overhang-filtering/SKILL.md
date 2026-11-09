---
name: read-overlap-overhang-filtering
description: Use when building an overlap graph or string graph from long noisy reads
  (Nanopore R9/R10, PacBio CLR) and partial overlaps with long unaligned overhangs
  would otherwise create spurious edges.
when_to_use_negative:
- Input reads are already high-accuracy (PacBio HiFi, Nanopore duplex) — the paper
  reports PECAT achieves only mediocre performance on these, where simpler assemblers
  like Hifiasm suffice
- Reads are short (Illumina-style) — minimap2 overlap presets '-x ava-pb'/'-x ava-ont'
  are tuned for long noisy reads with error rates up to ~15%
- The goal is reference-based mapping or polishing rather than de novo overlap graph
  construction
edam_operation: http://edamontology.org/operation_0525
edam_topics: []
tools:
- name: minimap2
  role: Detects candidate read-to-read overlaps using minimizer seeds (e.g. '-x ava-pb'
    for PacBio CLR) as the fast pre-filter before end extension
  repo: https://github.com/lh3/minimap2
- name: PECAT
  role: Implements the diff-algorithm end extension, overhang filtering, containment
    bridging, and Myers' transitive reduction that together limit local alignment
    to active string graph edges
  repo: https://github.com/lemene/PECAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:5bb27738792b00ad2568f5d2875af911f9eedf03c67415c94c45907a22674319
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Read overlap overhang filtering

## Summary

Detects candidate read-to-read overlaps with minimap2, extends partial alignments to read ends using the diff (edit-distance) algorithm, and filters overlaps that still retain long overhangs before string graph construction in PECAT.

## When to use

Use when building an overlap graph or string graph from long noisy reads (Nanopore R9/R10, PacBio CLR) and partial overlaps with long unaligned overhangs would otherwise create spurious edges.

## When NOT to use

- Input reads are already high-accuracy (PacBio HiFi, Nanopore duplex) — the paper reports PECAT achieves only mediocre performance on these, where simpler assemblers like Hifiasm suffice
- Reads are short (Illumina-style) — minimap2 overlap presets '-x ava-pb'/'-x ava-ont' are tuned for long noisy reads with error rates up to ~15%
- The goal is reference-based mapping or polishing rather than de novo overlap graph construction

## Inputs

- Corrected long reads (FASTA/FASTQ) from Nanopore R9, Nanopore R10 or PacBio CLR sequencing
- Read-to-read overlap candidates in PAF format from minimap2

## Outputs

- Filtered overlap set (overhang-free, contained reads handled) feeding string graph construction
- String graph with active/inactive edges after transitive reduction and identity-threshold filtering

## How to apply

First detect candidate overlaps between reads with minimap2 using overlap presets ('-x ava-pb' for PacBio CLR reads, '-x ava-ont' for Nanopore) under PECAT's align_rd2rd_options. Because minimap2 minimizer seed-and-extend alignments are typically interior and do not reach read ends, extend each candidate alignment to both read ends with the diff (edit-distance) algorithm. After extension, discard overlaps that still show long overhangs using align_filter_options, since genuine read overlaps should extend to read ends. In the subsequent string graph, contained reads are bridged and transitive overlaps are found with Myers' transitive reduction and marked inactive; low-quality edges whose identity falls below an adaptive threshold are removed, and broken paths are reconnected by reusing transitive edges and contained reads. This design means expensive local alignment (identity computation) is only performed on the small set of active edges, keeping assembly fast.

## Related tools

- **minimap2** (Detects candidate read-to-read overlaps using minimizer seeds (e.g. '-x ava-pb' for PacBio CLR) as the fast pre-filter before end extension) — https://github.com/lh3/minimap2
- **PECAT** (Implements the diff-algorithm end extension, overhang filtering, containment bridging, and Myers' transitive reduction that together limit local alignment to active string graph edges) — https://github.com/lemene/PECAT

## Examples

```
./minimap2 -x ava-pb reads.fa reads.fa > overlaps.paf
```

## Evaluation signals

- Surviving overlaps should align to (or very near) the ends of both reads after diff-algorithm extension; overlaps retaining long overhangs must have been dropped by align_filter_options
- Contained reads should be absent from the final graph as nodes (bridged through their containing reads), and transitive edges should be marked inactive rather than locally aligned
- Identity computation (local alignment) should only run on the small active-edge set — runtime should scale far below full pairwise alignment of all overlaps
- Edges whose identity falls below the adaptive threshold must be removed, and paths broken by their removal must be reconnected via transitive edges and contained reads
- Downstream PECAT demo validation: $PROJECT/3-assemble/primary.fasta is produced as a valid FASTA file

## Limitations

- Overhang filtering and identity thresholds operate on corrected reads; the paper notes that even after error correction, SNP/variant signal on corrected reads alone can be inaccurate in low heterozygosity regions
- PECAT needs higher data coverage to obtain high-quality phased assemblies
- minimap2 indexing parameters (-k, -w, -H, -I) cannot be changed once an index is built, so different data types may require separate indexes

## Evidence

- [discussion] we extend the alignment to the ends of the reads with the diff36 algorithm and filter out overlaps still with long overhangs: "we extend the alignment to the ends of the reads with the diff36 algorithm and filter out overlaps still with long overhangs"
- [discussion] we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [discussion] The edges whose identities are less than the threshold are marked as low quality and removed.: "The edges whose identities are less than the threshold are marked as low quality and removed."
- [readme] PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method.: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
