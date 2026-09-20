---
name: base-quality-weighted-graph-vertex-construction
description: Use when calling SVs from PE or LR BAM/CRAM data and you need per-breakpoint
  consensus contigs assembled from supporting read bases, or when benchmarking the
  linear-time behavior of the consensus stage.
when_to_use_negative:
- The goal is full de novo assembly of SVs or resolution of multi-allelic sites and
  full-length insertions in PE data — the paper notes these remain unresolved because
  dysgu has no de novo assembly stage.
- Input reads are already consensus contigs or variant calls (VCF) rather than per-base
  aligned supporting reads, since vertex construction requires individual supporting
  bases with base qualities.
- Only read-support information (no sequence) is wanted — the README's '--contigs
  False' model removes all sequence-related metrics and bypasses this stage.
tools:
- name: dysgu
  role: Provides the positional de Bruijn graph-inspired consensus sequence algorithm;
    its python-API (DysguSV, dysgu_default_args) is used to construct base-quality-weighted
    vertices and back-trace the longest path to emit consensus contigs.
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:ee355a174f0c02b667298a5760a152484baaee92c0a2ea56830671939464d12d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# base-quality-weighted-graph-vertex-construction

## Summary

Reconstructs dysgu's consensus-sequence generator for SV breakpoints: a positional de Bruijn graph-inspired DAG whose vertices are individual bases weighted by summed base quality, resolved to a consensus by longest-path back-tracing with soft-clip trimming.

## When to use

Use when calling SVs from PE or LR BAM/CRAM data and you need per-breakpoint consensus contigs assembled from supporting read bases, or when benchmarking the linear-time behavior of the consensus stage.

## When NOT to use

- The goal is full de novo assembly of SVs or resolution of multi-allelic sites and full-length insertions in PE data — the paper notes these remain unresolved because dysgu has no de novo assembly stage.
- Input reads are already consensus contigs or variant calls (VCF) rather than per-base aligned supporting reads, since vertex construction requires individual supporting bases with base qualities.
- Only read-support information (no sequence) is wanted — the README's '--contigs False' model removes all sequence-related metrics and bypasses this stage.

## Inputs

- SV-associated reads extracted by 'dysgu fetch' (input.dysgu_reads.bam) — split/discordant reads and reads with soft-clips >= 15 bp
- Sorted, indexed .bam/.cram of paired-end (bwa-mem) or long-read (minimap2/ngmlr) alignments
- Indexed reference genome FASTA
- Working directory for dysgu temporary files

## Outputs

- Per-breakpoint consensus contigs written to consensus_contigs.fasta
- Contigs spanning their breakpoint with low-quality (P_q < 0.5) soft-clipped ends trimmed
- Runtime-versus-input-bases scaling curve (runtime_scaling_curve.png) demonstrating O(V+E) behavior

## How to apply

Load the dysgu package and expose its consensus-generation API surface (e.g. DysguSV, dysgu_default_args). At each breakpoint locus, build base-tuple vertices (b_i, i, f, c) from supporting read bases, weighting each vertex by the summed base quality of its supporting bases; dysgu generates these supporting reads from alignment gaps, discordant/split reads and soft-clips >= 15 bp by default (paired-end). Order vertices along the genomic positions of the supporting mapper alignments into a weighted DAG, topologically sort it, and run breadth-first longest-path scoring with back-trace to recover the highest-quality consensus base per position. Trim soft-clipped bases at contig ends where path quality P_q < 0.5 and write per-breakpoint consensus contigs to consensus_contigs.fasta. Benchmark runtime over increasing numbers of input bases to confirm the expected O(V+E) linear scaling.

## Related tools

- **dysgu** (Provides the positional de Bruijn graph-inspired consensus sequence algorithm; its python-API (DysguSV, dysgu_default_args) is used to construct base-quality-weighted vertices and back-trace the longest path to emit consensus contigs.) — https://github.com/kcleal/dysgu

## Examples

```
dysgu run reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- Runtime-vs-input-bases curve is consistent with linear O(V+E) scaling across benchmarked input sizes.
- Every emitted contig in consensus_contigs.fasta spans its breakpoint.
- Soft-clipped bases at contig ends are trimmed wherever path quality P_q < 0.5.
- Each consensus base corresponds to the highest-quality (summed base-quality weighted) vertex along the back-traced longest path in the DAG.
- Contigs are regenerated from the SV-associated reads file (input.dysgu_reads.bam) produced by 'dysgu fetch'.

## Limitations

- Low-accuracy reads (e.g. older ONT) can cause reads to be filtered during the divergence check (default 0.02); use '--divergence auto' to avoid reduced sensitivity upstream of consensus generation.
- The consensus is 'inspired by' a positional de Bruijn graph and is per-breakpoint, not a de novo assembly stage — multi-allelic sites and full-length insertions in PE data remain unresolved.
- Computed insertion sizes from contigs are expected to be less accurate than de novo assembly-based callers such as manta and gatk (per the paper, 'data not shown').

## Evidence

- [intro] Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs.: "Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences"
- [other] Dysgu's consensus algorithm represents graph vertices as individual bases and reads out the consensus sequence by back-tracing through the longest path in the graph.: [evidence span withheld — anchor status: not_found]
- [readme] reads with a soft-clip >= clip_length (15 bp by default for paired-end reads): [evidence span withheld — anchor status: external_doc]
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
