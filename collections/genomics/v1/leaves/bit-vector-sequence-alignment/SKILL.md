---
name: bit-vector-sequence-alignment
description: Use when nanopore (MinION 1d/2d) or PacBio reads must be gapped-aligned
  to a reference after candidate mapping locations have been ranked, and BLAST-like
  sensitivity with >95% precision is required.
when_to_use_negative:
- When a lightweight coarse mapping (seed hits or anchors) is sufficient and full
  gapped alignment cost is unnecessary
- 'When index memory is a hard constraint: GraphMap''s hash index is ~14x reference
  size (~42 GB peak for a human genome), unlike BWT-based mappers'
- When reads must be mapped to a transcriptome with spliced alignments — a dedicated
  spliced-alignment mode was still under development
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Implements the five-stage funnel ending in Myers bit-vector (semiglobal) gapped
    alignment of long error-prone reads to a reference
  repo: https://github.com/isovic/graphmap
- name: marginAlign
  role: Downstream realigner that can consume GraphMap mappings as a better starting
    point for SNV calling
  repo: https://github.com/isovic/marginAlign
- name: graphmap-reproduce
  role: Scripts to reproduce the paper's benchmarking of alignment precision and sensitivity
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:c56d236656f58903749ed1c7a3596a7dc76fc2624601519ba98ef0c42102c70a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - graphmap-reproduce
  - make
  - marginAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# bit-vector-sequence-alignment

## Summary

Final-stage gapped alignment of long, error-prone nanopore reads using Myers' bit-vector (semiglobal) alignment, applied to top-ranked candidate regions after GraphMap's LCSk chaining and L1-regression filtering produce high-precision (>95%) mappings.

## When to use

Use when nanopore (MinION 1d/2d) or PacBio reads must be gapped-aligned to a reference after candidate mapping locations have been ranked, and BLAST-like sensitivity with >95% precision is required.

## When NOT to use

- When a lightweight coarse mapping (seed hits or anchors) is sufficient and full gapped alignment cost is unnecessary
- When index memory is a hard constraint: GraphMap's hash index is ~14x reference size (~42 GB peak for a human genome), unlike BWT-based mappers
- When reads must be mapped to a transcriptome with spliced alignments — a dedicated spliced-alignment mode was still under development

## Inputs

- Nanopore MinION reads (FASTA/FASTQ) with potentially high error rates
- Reference genome sequence (FASTA)
- Stage II alignment anchors / candidate mapping paths from seed hits

## Outputs

- Gapped alignments in SAM format (graphmap_alignments.sam) with CIGAR strings
- Mapping quality values and BLAST-style E-values (ZE:f SAM tag) per alignment
- Ranked candidate mapping locations

## How to apply

After Stage II anchors are chained into candidate paths with a variable-length LCSk (longest common subsequence of k-mers) procedure, fit a 45-degree line to each candidate path by L1 (least absolute deviation) regression, discard anchors whose residuals fall outside the ±3-sigma confidence band, and re-run LCSk chaining on the retained anchors. Score the resulting candidate regions with the region quality scoring formulae (Eqs. 4-7) to rank the best candidate locations. Refine only the top-ranked candidates with Myers bit-vector alignment (GraphMap also offers Gotoh and anchored strategies) to produce the final gapped CIGAR of the read. Emit SAM output including mapping quality and a BLAST-style E-value (ZE:f tag); the E-value for unmapped reads must be a valid float, not Inf. The progressive read-funneling design is what allows bit-vector alignment to run fast while retaining sensitivity on high-error-rate reads.

## Related tools

- **GraphMap** (Implements the five-stage funnel ending in Myers bit-vector (semiglobal) gapped alignment of long error-prone reads to a reference) — https://github.com/isovic/graphmap
- **marginAlign** (Downstream realigner that can consume GraphMap mappings as a better starting point for SNV calling) — https://github.com/isovic/marginAlign
- **graphmap-reproduce** (Scripts to reproduce the paper's benchmarking of alignment precision and sensitivity) — https://github.com/isovic/graphmap-reproduce

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Emitted SAM file parses correctly with valid CIGAR strings and a float-valued ZE:f E-value tag (never Inf after v0.5.2)
- Mapping precision exceeds 95%, with location precision typically >98%
- Mapped bases exceed 95% of read bases on MinION data (10-80% increase versus other mappers)
- Alignments span indels of 100 bp to 4 kbp, enabling direct structural variant detection without split-read analysis
- Alignments are invariant to alignment-parameter choices ('mapping position agnostic to alignment parameters')

## Limitations

- Requires large-memory machines: ~100 GB for mapping to the human genome; memory-efficient (FM-index) alternative is only 'currently under implementation'
- Minimizer index and --freq-percentile 0.99 seed filtering slightly reduce sensitivity, especially on large references; use -x sensitive to disable
- Index files are incompatible across versions — rebuild with make modules after git pull or use --rebuild-index

## Evidence

- [readme] Various alignment strategies (semiglobal bit-vector and Gotoh, anchored).: [evidence span withheld — anchor status: external_doc]
- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] GraphMap's precision is typically >98% and it is able to distinguish between candidate locations: "GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome"
- [readme] Changed the unmapped / Inf E-value (the ZE:f tag) to be a valid float value: "Changed the unmapped / Inf E-value (the ZE:f tag) to be a valid float value (`std::numeric_limits<float>::max()` instead of `Inf`)."
