---
name: positional-de-brujin-graph-consensus-assembly
description: Use when generating consensus contigs at structural variant breakpoints
  from split/discordant/soft-clipped reads, or when reproducing or benchmarking dysgu's
  consensus stage and its linear-time O(V+E) scaling behaviour.
when_to_use_negative:
- Input is raw whole-genome BAM with no SV-associated reads extracted — dysgu's fetch
  stage must first separate split/discordant/soft-clipped reads before consensus assembly
- Goal is full de novo assembly of multi-allelic sites or full-length insertion sequences
  from paired-end data — the paper notes dysgu lacks a de novo assembly stage and
  these remain unresolved
- Reads are low-accuracy and no --divergence setting is considered — the default divergence
  of 0.02 is too stringent for lower-accuracy reads and will over-filter reads feeding
  the graph
edam_operation: http://edamontology.org/operation_0523
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: dysgu
  role: Provides the positional de Bruijn graph-inspired consensus sequence algorithm
    and the python API (DysguSV, dysgu_default_args) used to generate and benchmark
    consensus contigs at SV breakpoints
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:ad75b21c612aea656299f4559b08e24726f33bc59743b54165ad0a6f46fe2fb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_0523
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# positional-de-brujin-graph-consensus-assembly

## Summary

Reconstructs dysgu's fast consensus-sequence generator, which builds a positional de Bruijn graph-inspired DAG whose vertices are individual bases weighted by summed base quality, then back-traces the longest path to emit a consensus contig per SV breakpoint for downstream remapping.

## When to use

Use when generating consensus contigs at structural variant breakpoints from split/discordant/soft-clipped reads, or when reproducing or benchmarking dysgu's consensus stage and its linear-time O(V+E) scaling behaviour.

## When NOT to use

- Input is raw whole-genome BAM with no SV-associated reads extracted — dysgu's fetch stage must first separate split/discordant/soft-clipped reads before consensus assembly
- Goal is full de novo assembly of multi-allelic sites or full-length insertion sequences from paired-end data — the paper notes dysgu lacks a de novo assembly stage and these remain unresolved
- Reads are low-accuracy and no --divergence setting is considered — the default divergence of 0.02 is too stringent for lower-accuracy reads and will over-filter reads feeding the graph

## Inputs

- SV-associated reads (split reads, discordant pairs, and reads with soft-clip >= 15 bp by default for paired-end data) from a sorted/indexed BAM/CRAM
- Indexed reference genome FASTA
- Aligned BAM/CRA produced by bwa mem (paired-end) or minimap2/ngmlr (PacBio/ONT long reads)
- Supporting read base qualities (per-base Q values) used to weight graph vertices

## Outputs

- Per-breakpoint consensus contigs written to consensus_contigs.fasta, with soft-clipped ends (P_q < 0.5) trimmed
- Consensus sequences passed to dysgu's remapping step and to the machine-learning quality classifier as contig-derived features
- runtime_scaling_curve.png plotting consensus runtime versus number of input bases

## How to apply

Load the dysgu python API (dysgu 1.x, e.g. DysguSV / dysgu_default_args from the documented module) and, at each SV breakpoint, construct base-tuple vertices (b_i, i, f, c) from the supporting read bases, weighting each vertex by the summed base quality of its supporting bases. Order vertices by the genomic positions of the supporting mapper alignments to form a weighted DAG, topologically sort it, and run a breadth-first longest-path scoring with backtrace so each position of the consensus contributes its highest-quality supporting base. Trim soft-clipped bases at contig ends where the accumulated path quality P_q falls below 0.5, then write the per-breakpoint consensus contigs to consensus_contigs.fasta for use in dysgu's subsequent remapping and ML quality-classification stages. Benchmark the routine over increasing numbers of input bases to confirm linear O(V+E) runtime scaling, and validate that every emitted contig spans its breakpoint with low-quality (P_q < 0.5) soft-clip bases removed.

## Related tools

- **dysgu** (Provides the positional de Bruijn graph-inspired consensus sequence algorithm and the python API (DysguSV, dysgu_default_args) used to generate and benchmark consensus contigs at SV breakpoints) — https://github.com/kcleal/dysgu

## Examples

```
dysgu call --ibam all_reads.bam reference.fa temp_dir temp_dir/temp_dir.dysgu_reads.bam > svs.vcf
```

## Evaluation signals

- Every emitted contig spans its breakpoint with low-quality (P_q < 0.5) soft-clipped bases trimmed from contig ends
- Runtime-versus-input-bases benchmark curve is consistent with linear O(V+E) scaling across the benchmarked input sizes
- Each consensus position corresponds to the highest summed base-quality supporting base recovered by the longest-path backtrace
- Downstream remapping of the consensus contigs recovers additional small SVs beyond the initial cigar/discordant/split-read signal detection
- Running `dysgu test` completes without error after installation, confirming the built consensus machinery is functional

## Limitations

- Consensus insertions sizes in paired-end data are expected to be less accurate than de novo assembly-based callers such as manta and gatk ('data not shown')
- The consensus stage feeds contig metrics to the ML classifier, but the feature list is non-exhaustive and heuristics (span position distance, minimizer support) are unoptimized and could be tuned with a larger test set
- Without an assembly stage, multi-allelic sites and full-length insertion sequences in PE data remain unresolved
- The default sequence divergence of 0.02 can be too stringent for lower-accuracy reads, filtering more reads and lowering sensitivity; '--divergence auto' is recommended for noisy long reads

## Evidence

- [intro] Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs.: "Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs."
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning"
- [readme] the 'run' pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads): "The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads,
and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads)."
- [readme] the default is set at 0.02 which can be too stringent for lower accuracy reads and will result in more reads being filtered and lower sensitivity: "The default is set at 0.02 which can be too stringent for lower accuracy reads and will result in
more reads being filtered and lower sensitivity"
- [discussion] Putative events are then labelled with a useful probability value using a gradient boosting classifier (30).: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
