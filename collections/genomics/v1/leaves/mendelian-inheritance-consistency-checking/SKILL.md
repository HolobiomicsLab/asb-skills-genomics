---
name: mendelian-inheritance-consistency-checking
description: Use when genotyping tandem repeats from haplotype-resolved long-read
  alignments in a trio/pedigree and you need to classify each TR call as Mendelian
  consistent or inconsistent, or estimate the overall consistency fraction.
when_to_use_negative:
- Input lacks trio/pedigree structure (no both parents sequenced) — Mendelian consistency
  cannot be traced
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR
  cannot identify
- Samples are polyploid or on haploid chromosomes (e.g., human Y chromosome), which
  TRiCoLOR does not support
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: Parent tool providing the --mendel option to trace Mendelian consistency of
    TR genotypes in pedigrees
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SAGE
  role: Genotyping module that collects spanning reads per haplotype per parent, builds
    SPOA consensus, and emits multi-sample BCF with Mendelian flags
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: Partial order alignment used to compute haplotype-specific low-error consensus
    sequences (~88% error reduction on PB data)
- name: minimap2
  role: Aligns parental consensus sequences to GRCh38 for reference-like vs child-TR-like
    genotyping by edit distance
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:62abdd22962c7da0150bb104a4fe5ce454f07c0b59a2a21a71ef3926f8cfd3cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR SAGE
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mendelian-inheritance-consistency-checking

## Summary

Trace Mendelian consistency of tandem repeat genotypes across a trio (e.g., the Puerto Rican PB trio HG00731/HG00732/HG00733) using TRiCoLOR SAGE's --mendel parameter, which flags each TR as Mendelian consistent or inconsistent given inferred parental genotypes.

## When to use

Use when genotyping tandem repeats from haplotype-resolved long-read alignments in a trio/pedigree and you need to classify each TR call as Mendelian consistent or inconsistent, or estimate the overall consistency fraction.

## When NOT to use

- Input lacks trio/pedigree structure (no both parents sequenced) — Mendelian consistency cannot be traced
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR cannot identify
- Samples are polyploid or on haploid chromosomes (e.g., human Y chromosome), which TRiCoLOR does not support

## Inputs

- Haplotype-resolved PacBio BAM alignments for HG00731 (father), HG00732 (mother), and HG00733 (child) from HGSVC
- GRCh38 reference genome
- Child TR region BED file produced by TRiCoLOR SENSoR/REFER

## Outputs

- TRiCoLOR SAGE multi-sample BCF with genotypes for child and both parents plus Mendelian-consistency flags
- Per-TR genotyping/Mendelian-consistency TSV table
- Overall fraction of Mendelian-consistent TRs (expected ~80% for the PB trio under the ≥10 bp filter)

## How to apply

Run TRiCoLOR SAGE on the child TR region BED (from SENSoR/REFER) to collect spanning reads per haplotype per parent, trim them to the region, and build haplotype-specific low-error consensus sequences with SPOA (~88% error reduction on PacBio data). Align parental consensus sequences to GRCh38 with minimap2 and genotype each parental TR as reference-like or child-TR-like by comparing edit distances. Enable the --mendel parameter to flag each TR as Mendelian consistent or inconsistent given the inferred parental genotypes, and export the multi-sample BCF converted to a per-TR genotyping/Mendelian-consistency TSV. Restrict the consistency assessment to variants differing from the reference by ≥10 bp (including multi-allelic variants differing by the same distance), then compute the overall fraction of Mendelian-consistent TRs, which should approximate the ~80% reported for the PB trio analysis.

## Related tools

- **TRiCoLOR** (Parent tool providing the --mendel option to trace Mendelian consistency of TR genotypes in pedigrees) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SAGE** (Genotyping module that collects spanning reads per haplotype per parent, builds SPOA consensus, and emits multi-sample BCF with Mendelian flags) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (Partial order alignment used to compute haplotype-specific low-error consensus sequences (~88% error reduction on PB data))
- **minimap2** (Aligns parental consensus sequences to GRCh38 for reference-like vs child-TR-like genotyping by edit distance)

## Examples

```
tricolor SAGE --bams HG00731.bam HG00732.bam HG00733.bam --bed child_TRs.bed --reference GRCh38.fa --mendel --output sage_trio.bcf
```

## Evaluation signals

- The per-TR table contains genotype calls for the index child and both parents
- The overall Mendelian-consistent fraction approximates the ~80% rate reported for the PB trio under the ≥10 bp reference-difference filter
- The Mendelian-consistency assessment is restricted to variants differing from the reference by ≥10 bp (including multi-allelic variants differing by the same distance)
- Output is a valid multi-sample BCF convertible to a per-TR TSV with Mendelian-consistency annotations

## Limitations

- Filtering for variants differing from the reference by ≥10 bp is required for meaningful consistency estimates; unfiltered calls are noisy
- TRiCoLOR cannot identify repeats in unassembled genome regions (e.g., human centromeres and telomeres)
- No support for polyploid species or haploid chromosomes (e.g., human Y chromosome)

## Evidence

- [abstract] The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees.: "The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees."
- [methods] SPOA error reduction on PB data: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [results] Mendelian consistency filtering thresholds: "Filtering for variants differing from the reference for ≥10 bp and for multi-allelic variants differing from each other by the same distance"
- [intro] Trio workflow step: "For trio sequencing studies, TRiCoLOR additionally allows Mendelian inheritance patterns to be traced across TR genotypes."
- [discussion] Unassembled region limitation: "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
