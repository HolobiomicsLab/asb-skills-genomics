---
name: pedigree-trio-genotype-analysis
description: Use when a parent–parent–child trio of haplotype-resolved long-read alignments
  is available and you need to flag each genotyped tandem repeat as Mendelian consistent
  or inconsistent, as done for the HG00731/HG00732/HG00733 Puerto Rican PB trio.
when_to_use_negative:
- Samples are not a parent–parent–child trio pedigree, since Mendelian-consistency
  tracing requires inferred genotypes for both parents and the index child
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR
  cannot identify
- The organism is polyploid or the target is a haploid chromosome (e.g., the human
  Y chromosome), which TRiCoLOR does not support
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: TRiCoLOR
  role: Trio-aware tandem repeat profiler; its --mendel parameter flags each TR as
    Mendelian consistent or inconsistent from the inferred parental genotypes
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SAGE
  role: 'Genotyping module: collects spanning reads per haplotype per parent, trims
    to the TR region, and emits the multi-sample BCF of trio genotypes'
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: Computes haplotype-specific low-error consensus sequences from the parental
    spanning reads (~88% error reduction on PB data)
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
  source_content_hash: sha256:15aa59f60ca8454d3d1f1af779f88d340f5699e5738e0e33b516fc82250c133f
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# pedigree-trio-genotype-analysis

## Summary

Traces Mendelian consistency of tandem repeat genotypes across a pedigree trio using TRiCoLOR SAGE's --mendel flag, comparing child TR alleles against haplotype-specific parental genotypes inferred from PB long-read consensus sequences.

## When to use

Use when a parent–parent–child trio of haplotype-resolved long-read alignments is available and you need to flag each genotyped tandem repeat as Mendelian consistent or inconsistent, as done for the HG00731/HG00732/HG00733 Puerto Rican PB trio.

## When NOT to use

- Samples are not a parent–parent–child trio pedigree, since Mendelian-consistency tracing requires inferred genotypes for both parents and the index child
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR cannot identify
- The organism is polyploid or the target is a haploid chromosome (e.g., the human Y chromosome), which TRiCoLOR does not support

## Inputs

- Haplotype-resolved Pacific Biosciences BAM alignments for HG00731 (father), HG00732 (mother), HG00733 (child) from HGSVC
- GRCh38 reference genome
- Child TR region BED file produced by TRiCoLOR SENSoR/REFER

## Outputs

- TRiCoLOR SAGE multi-sample BCF with genotypes for the child and both parents plus Mendelian-consistency flags per TR
- Per-TR genotyping/Mendelian-consistency TSV
- Overall fraction of Mendelian-consistent TRs (~80% expected for the PB trio)

## How to apply

Run TRiCoLOR SAGE on the child TR region BED (from SENSoR/REFER) against GRCh38, with haplotype-resolved PB BAM alignments for the father (HG00731), mother (HG00732), and child (HG00733): SAGE collects spanning reads per haplotype per parent, trims them to the TR region, and computes haplotype-specific low-error consensus sequences with SPOA (~88% error reduction on PB data). Each parental consensus is then aligned to GRCh38 with minimap2 and genotyped as reference-like or child-TR-like by comparing edit distances. Enable the --mendel parameter so each TR is flagged as Mendelian consistent or inconsistent given the inferred parental genotypes, and restrict the consistency assessment to variants differing from the reference by ≥10 bp (including multi-allelic variants differing by the same distance). Export the SAGE multi-sample BCF, convert to a per-TR TSV of genotypes and Mendelian flags, and compute the overall fraction of Mendelian-consistent TRs, which should approximate the ~80% reported for the PB trio.

## Related tools

- **TRiCoLOR** (Trio-aware tandem repeat profiler; its --mendel parameter flags each TR as Mendelian consistent or inconsistent from the inferred parental genotypes) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SAGE** (Genotyping module: collects spanning reads per haplotype per parent, trims to the TR region, and emits the multi-sample BCF of trio genotypes) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (Computes haplotype-specific low-error consensus sequences from the parental spanning reads (~88% error reduction on PB data))
- **minimap2** (Aligns parental consensus sequences to GRCh38 for reference-like vs child-TR-like genotyping by edit distance)

## Examples

```
tricolor SAGE --bam child.bam father.bam mother.bam --regions child_trs.bed --reference GRCh38.fa --mendel --output sage_trio
```

## Evaluation signals

- Per-TR TSV contains genotype calls for the index child and both parents (multi-sample output present)
- Overall Mendelian-consistent fraction approximates the ~80% rate reported in the paper for the HG00733 PB trio
- Mendelian flags are restricted to variants differing from the reference by ≥10 bp, including multi-allelic variants differing by the same distance
- SAGE output is a valid multi-sample BCF convertible to the per-TR TSV
- Parental consensus-based genotypes reflect edit-distance comparison against GRCh38 and the child TR as described

## Limitations

- Mendelian-consistency statistics only become interpretable after filtering for variants differing from the reference by ≥10 bp (and multi-allelic variants differing by the same distance)
- TRiCoLOR does not support polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected, so they are absent from the consistency assessment
- Parameter values for profiling mini-satellites are not validated; the tool has been extensively applied only to micro-satellites

## Evidence

- [intro] For trio sequencing studies, TRiCoLOR additionally allows Mendelian inheritance patterns to be traced across TR genotypes.: "For trio sequencing studies, TRiCoLOR additionally allows Mendelian inheritance patterns to be traced across TR genotypes."
- [results] Filtering for variants differing from the reference for ≥10 bp and for multi-allelic variants differing from each other by the same distance: "Filtering for variants differing from the reference for ≥10 bp and for multi-allelic variants differing from each other by the same distance"
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [abstract] The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees.: "The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees."
- [discussion] future work includes extending its feature set to polyploid species and haploid chromosomes (human Y chromosome): "future work includes extending its feature set to polyploid species and haploid chromosomes (human Y chromosome)"
