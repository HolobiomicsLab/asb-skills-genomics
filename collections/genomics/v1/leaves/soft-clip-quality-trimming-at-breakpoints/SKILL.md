---
name: soft-clip-quality-trimming-at-breakpoints
description: Use when generating or auditing consensus contigs at SV breakpoints from
  split/discordant/soft-clipped reads, or when verifying that low-quality clipped
  bases are trimmed before contig remapping.
when_to_use_negative:
- The input is a final VCF of SV calls rather than raw SV-associated alignments —
  this skill operates on the pre-calling consensus-generation stage, not on filtering
  or merging calls.
- A full de novo assembly stage is required (e.g. resolving multi-allelic sites or
  full-length insertion sequences in PE data) — the article notes dysgu lacks such
  a stage and consensus-based insertion sizes are expected to be less accurate than
  assembly-based callers such as manta and gatk.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the consensus sequence API (DysguSV, dysgu_default_args), the fetch
    stage that isolates SV-associated reads, and the linear-time positional de Bruijn
    graph-inspired consensus algorithm whose outputs are trimmed and written as consensus
    contigs
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:dfb66098363d5a00c6f5d9d6e7e447ea5d639b59335abd8845b129afc159d599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# soft-clip-quality-trimming-at-breakpoints

## Summary

Reconstructs dysgu's linear-time consensus sequence generator for structural variant breakpoints: base-tuple graph vertices weighted by summed base quality, longest-path back-tracing to emit per-breakpoint consensus contigs, and trimming of soft-clipped bases where path quality P_q < 0.5.

## When to use

Use when generating or auditing consensus contigs at SV breakpoints from split/discordant/soft-clipped reads, or when verifying that low-quality clipped bases are trimmed before contig remapping.

## When NOT to use

- The input is a final VCF of SV calls rather than raw SV-associated alignments — this skill operates on the pre-calling consensus-generation stage, not on filtering or merging calls.
- A full de novo assembly stage is required (e.g. resolving multi-allelic sites or full-length insertion sequences in PE data) — the article notes dysgu lacks such a stage and consensus-based insertion sizes are expected to be less accurate than assembly-based callers such as manta and gatk.

## Inputs

- SV-associated reads from a sorted/indexed BAM or CRAM (split/discordant reads and reads with soft-clip >= 15 bp for paired-end data, as produced by dysgu fetch into input.dysgu_reads.bam)
- Indexed reference genome FASTA
- Breakpoint loci identified from alignment cigar information, alignment gaps, and discordant/supplementary mappings

## Outputs

- Per-breakpoint consensus contigs written to consensus_contigs.fasta, with soft-clipped bases at contig ends trimmed where path quality P_q < 0.5
- Runtime-versus-input-bases benchmark curve (runtime_scaling_curve.png) used to confirm O(V+E) linear scaling

## How to apply

Expose dysgu's consensus-generation API surface (DysguSV, dysgu_default_args) from the documented module. At each breakpoint locus, construct base-tuple vertices (b_i, i, f, c) from supporting read bases, weighting each vertex by the summed base quality of its supporting bases, and order vertices along the genomic positions of the supporting mapper alignments to build a weighted DAG. Topologically sort the DAG and run breadth-first longest-path scoring with back-tracing so the highest-quality consensus base is recovered per position. Trim soft-clipped bases at contig ends where path quality P_q < 0.5, then write the per-breakpoint consensus contigs to consensus_contigs.fasta. Rationale: dysgu's algorithm is inspired by the positional de Bruijn graph and runs in linear time (O(V+E)), so runtime should scale linearly with the number of input bases — benchmark with increasing input-base counts to confirm.

## Related tools

- **dysgu** (Provides the consensus sequence API (DysguSV, dysgu_default_args), the fetch stage that isolates SV-associated reads, and the linear-time positional de Bruijn graph-inspired consensus algorithm whose outputs are trimmed and written as consensus contigs) — https://github.com/kcleal/dysgu

## Examples

```
dysgu run -p4 reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- Every emitted contig spans its breakpoint with low-quality (P_q < 0.5) soft-clipped bases trimmed at the contig ends
- The runtime-versus-input-bases curve is consistent with linear O(V+E) scaling across the benchmarked input sizes
- Each consensus base corresponds to the back-traced longest path through the quality-weighted DAG (highest summed base quality per position)
- Input reads match the fetch-stage definition: split/discordant reads or reads with soft-clip >= clip_length (15 bp default for paired-end)

## Limitations

- Dysgu has no de novo assembly stage, so multi-allelic sites and full-length insertion sequences in paired-end data remain unresolved; consensus-based insertion sizes are expected to be less accurate than assembly-based callers such as manta and gatk
- The default sequence divergence of 0.02 can be too stringent for lower-accuracy long reads, filtering more reads and lowering sensitivity; '--divergence auto' is recommended for higher-error reads
- When fetch is used, a large temp file of SV-associated reads (>5 Gb for a 30X genome) is generated, and the stage assumes vertices are correctly ordered along the supporting alignments' genomic positions

## Evidence

- [intro] Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs.: "Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs."
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning"
- [other] Dysgu's consensus algorithm represents graph vertices as individual bases and reads out the consensus sequence by back-tracing through the longest path in the graph.: [evidence span withheld — anchor status: not_found]
- [readme] All SV associated reads will be placed in `samp1_temp/input.dysgu_reads.bam`.: [evidence span withheld — anchor status: external_doc]
