---
name: dag-topological-sort-longest-path-backtrace
description: Reference for reproducing or re-implementing dysgu's consensus contig
  stage — when you need per-breakpoint consensus sequences from supporting split/discordant
  read bases with quality-weighted path selection and soft-clip trimming.
when_to_use_negative:
- You need full-length insertion sequences or multi-allelic site resolution from PE
  data — the article notes no de novo assembly stage is incorporated, so consensus
  contigs alone cannot resolve these
- Your goal is de novo whole-genome assembly; this is a targeted per-breakpoint consensus
  of SV-associated reads, not an assembler
- The input BAM has no SV-associated signal (no split/discordant reads or soft-clips)
  — there will be no supporting bases to build the DAG from
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Source of the consensus-generation API (DysguSV, dysgu_default_args); the
    positional de Bruijn graph-inspired consensus algorithm being reconstructed
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Optional downstream benchmarking of the SV calls whose consensus contigs this
    stage produces
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:16fb33d6327f4bc470ee71b81b64c5fd282d8912a0950d4d9b0880659b8f8071
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - svbench
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# dag-topological-sort-longest-path-backtrace

## Summary

Reconstructs dysgu's linear-time consensus sequence generator at SV breakpoints: base-tuple vertices (b_i, i, f, c) weighted by summed base quality are ordered into a weighted DAG, topologically sorted, and solved by breadth-first longest-path scoring with backtrace, emitting per-breakpoint consensus contigs.

## When to use

Reference for reproducing or re-implementing dysgu's consensus contig stage — when you need per-breakpoint consensus sequences from supporting split/discordant read bases with quality-weighted path selection and soft-clip trimming.

## When NOT to use

- You need full-length insertion sequences or multi-allelic site resolution from PE data — the article notes no de novo assembly stage is incorporated, so consensus contigs alone cannot resolve these
- Your goal is de novo whole-genome assembly; this is a targeted per-breakpoint consensus of SV-associated reads, not an assembler
- The input BAM has no SV-associated signal (no split/discordant reads or soft-clips) — there will be no supporting bases to build the DAG from

## Inputs

- Sorted, indexed .bam/.cram of paired-end (bwa mem) or long-read (minimap2/ngmlr) alignments
- Indexed reference genome FASTA (e.g. reference.fa)
- SV-associated reads separated by dysgu fetch into input.dysgu_reads.bam (split/discordant reads, soft-clips >= 15 bp)
- Supporting read bases and base qualities at each breakpoint locus

## Outputs

- Per-breakpoint consensus contigs written to consensus_contigs.fasta
- Runtime-versus-input-bases scaling curve (runtime_scaling_curve.png)
- Consensus contigs that feed dysgu's sequence-related metrics and machine-learning quality score

## How to apply

Load the dysgu package and expose its consensus-generation API surface (DysguSV, dysgu_default_args). At each breakpoint locus, build vertices as base tuples (b_i, i, f, c) from supporting read bases, weighting each vertex by the summed base quality of its supporting bases. Order vertices along the genomic positions of the supporting mapper alignments to form a weighted DAG, topologically sort it, and run breadth-first longest-path scoring with backtrace to recover the highest-quality consensus base per position — the algorithm mirrors a positional de Bruijn graph. Trim soft-clipped bases at contig ends where path quality P_q < 0.5, then write per-breakpoint consensus contigs to consensus_contigs.fasta. Benchmark over increasing numbers of input bases to confirm O(V+E) scaling.

## Related tools

- **dysgu** (Source of the consensus-generation API (DysguSV, dysgu_default_args); the positional de Bruijn graph-inspired consensus algorithm being reconstructed) — https://github.com/kcleal/dysgu
- **svbench** (Optional downstream benchmarking of the SV calls whose consensus contigs this stage produces) — https://github.com/kcleal/svbench

## Examples

```
from dysgu import DysguSV, dysgu_default_args  # expose consensus-generation API surface per task_007 workflow
```

## Evaluation signals

- Runtime-versus-input-bases curve is consistent with linear O(V+E) scaling across the benchmarked input sizes
- Every emitted contig in consensus_contigs.fasta spans its breakpoint
- All soft-clipped bases with path quality P_q < 0.5 are trimmed from contig ends
- Each consensus base at a position is the backtraced highest-quality (summed supporting base quality) vertex in the topologically sorted DAG
- Consensus contigs reproduce dysgu's sequence-related metrics when run through DysguSV with default (diploid) arguments

## Limitations

- Dysgu relies on several unoptimized heuristics (e.g. span position distance, minimizer support) that could shift which reads contribute vertices
- Consensus-based insertion size accuracy is weaker than de novo assembly-based callers such as manta and gatk
- Without --divergence auto on higher-error reads, the default divergence of 0.02 filters more reads, reducing the vertex set available for consensus construction

## Evidence

- [intro] Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs.: "Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs."
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [readme] This will first run the `fetch` command that will create a temporary bam file plus other analysis files: [evidence span withheld — anchor status: external_doc]
