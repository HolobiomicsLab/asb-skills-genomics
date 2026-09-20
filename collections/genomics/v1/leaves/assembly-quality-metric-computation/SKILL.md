---
name: assembly-quality-metric-computation
description: Use when assemblies have been produced at multiple long-read coverage
  levels (e.g., 5x-45x PacBio) and you need per-level NGA50 and genome fraction values,
  aligned to a reference genome, to identify coverage plateaus.
when_to_use_negative:
- No reference genome is available, so reference-based NGA50 and genome fraction cannot
  be computed
- Input is still raw reads (FASTQ) rather than assembled contigs; assemble first with
  haslr.py before scoring
- The goal is speed/memory benchmarking of the assembler itself rather than contiguity/accuracy
  metrics
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0108
- http://edamontology.org/topic_3168
tools:
- name: HASLR
  role: Produces the hybrid assemblies (asm.final.fa) whose NGA50 and genome fraction
    are evaluated at each long-read coverage level
  repo: https://github.com/vpc-ccg/haslr
- name: minimap2
  role: Aligns the evaluated contigs to the reference genome (e.g., intra-species
    assembly-to-reference alignment) to derive QUAST-style metrics
  repo: https://github.com/lh3/minimap2
- name: QUAST
  role: Assembly-quality evaluator used to compute NGA50 and genome fraction per coverage
    level
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:a89a735710c4ea6ed72fea25d7d3c4084937ca57d1d507120ab5763b9f4a964f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - HASLR
  - QUAST
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0108
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# assembly-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes reference-based assembly quality metrics (NGA50 and genome fraction) for hybrid assemblies generated across long-read coverage levels, as in the HASLR Figure 4 coverage sensitivity benchmark, to determine where contiguity gains plateau.

## When to use

Use when assemblies have been produced at multiple long-read coverage levels (e.g., 5x-45x PacBio) and you need per-level NGA50 and genome fraction values, aligned to a reference genome, to identify coverage plateaus.

## When NOT to use

- No reference genome is available, so reference-based NGA50 and genome fraction cannot be computed
- Input is still raw reads (FASTQ) rather than assembled contigs; assemble first with haslr.py before scoring
- The goal is speed/memory benchmarking of the assembler itself rather than contiguity/accuracy metrics

## Inputs

- Hybrid assembly contigs per coverage level (asm.final.fa from haslr.py runs with -g 4.6m)
- Reference genome FASTA for E. coli MG1655
- Coverage labels / subsampled PacBio long-read FASTQ files (ecoli_filtered.fastq.gz subsets at 5x-45x)

## Outputs

- coverage_sweep_results.csv tabulating NGA50 and genome fraction versus long-read coverage (5x-45x)
- Per-assembly alignments of asm.final.fa against the MG1655 reference
- Plots of NGA50 and genome fraction curves across the 5x-45x coverage sweep

## How to apply

Collect each assembly output (asm.final.fa) produced by haslr.py runs at target coverages in {5x, 10x, 15x, 20x, 25x, 30x, 35x, 40x, 45x}, where long reads were subsampled longest-reads-first following HASLR's internal --cov-lr strategy (default longest 25x). Align each assembly to the E. coli MG1655 reference using an assembly-quality evaluator with QUAST-style metrics, computing NGA50 and genome fraction per coverage level. Tabulate both metrics versus coverage in coverage_sweep_results.csv and plot both curves across 5x-45x. Judge correctness by whether the largest NGA50 increase falls between 20x and 30x coverage, NGA50 changes above 30x are not significant, and genome fraction plateaus from 15x onward, matching the paper's coverage benchmark.

## Related tools

- **HASLR** (Produces the hybrid assemblies (asm.final.fa) whose NGA50 and genome fraction are evaluated at each long-read coverage level) — https://github.com/vpc-ccg/haslr
- **minimap2** (Aligns the evaluated contigs to the reference genome (e.g., intra-species assembly-to-reference alignment) to derive QUAST-style metrics) — https://github.com/lh3/minimap2
- **QUAST** (Assembly-quality evaluator used to compute NGA50 and genome fraction per coverage level)

## Examples

```
minimap2 -cx asm5 E_coli_MG1655.fa asm_30x/asm.final.fa > asm_30x_vs_ref.paf
```

## Evaluation signals

- coverage_sweep_results.csv contains one row per coverage level in {5x, 10x, 15x, 20x, 25x, 30x, 35x, 40x, 45x} with non-missing NGA50 and genome fraction
- The largest NGA50 increase occurs between 20x and 30x long-read coverage, and NGA50 changes above 30x are not significant
- Genome fraction plateaus from 15x coverage onward, matching the reported coverage benchmark trend
- Each assembly's aligned length is consistent with the 4.6 Mb genome size passed as -g 4.6m, with genome fraction below 100%
- Every asm.final.fa aligns to the MG1655 reference without assembly fragments failing to map due to misassemblies

## Limitations

- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 than other tools (e.g., Canu on the human genome), so absolute metric values may understate performance
- HASLR usually has a higher indel rate compared with other tools, which affects accuracy metrics even when contiguity looks good
- Backbone graph connectivity depends on the existence of unique SR contigs; regions with high heterozygosity remain unresolved because Minia short read contigs are fragmented and filtered out, lowering genome fraction
- Read length is an important factor in assembly contiguity, so a coverage sweep alone may confound contiguity effects with read-length effects

## Evidence

- [readme] ./minimap2 -cx asm5 asm1.fa asm2.fa > aln.paf             # intra-species asm-to-asm alignment: [evidence span withheld — anchor status: external_doc]
