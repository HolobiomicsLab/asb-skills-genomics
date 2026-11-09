---
name: repetitive-element-annotation-of-sv-breakpoints
description: Use when you have a long-read (e.g., MinION NanoSV) SV VCF plus matched
  short-read call sets and want to characterize which novel SVs overlap repeats or
  retrotransposon insertions.
when_to_use_negative:
- The input call set is already intersected with short-read calls, so a novelty comparison
  would double-count shared variants
- The sequencing data are not human patient genomes with matched short-read data from
  the same individuals, since the shared-vs-specific classification requires both
  call sets on the same reference genome
- The goal is de novo complex rearrangement (e.g., chromothripsis) breakpoint resolution,
  which requires read-based phasing rather than repeat-content annotation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Produces the long-read SV VCF call sets that are then classified as shared
    or nanopore-specific and annotated for repeat content
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts and sorts LAST-generated SAM alignments into coordinate-sorted BAM
    files used as NanoSV input
  repo: https://github.com/biod/sambamba
- name: Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)
  role: Provide the short-read SV call sets against which nanopore calls are intersected
    to define the novel set
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:ed98ddcb968a077f9996a65d2469e6cea9e24b692f91f061ba5aac25586271fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FREEC
  - GATK HaplotypeCaller
  - Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)
  - Manta
  - Mobster
  - NanoSV
  - Pindel
  - sambamba
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Repetitive element annotation of SV breakpoints

## Summary

Annotates nanopore-derived structural variant call sets with repeat content from UCSC RepeatMasker and DFAM to flag retrotransposon/LINE insertions, and contrasts GC content and size distributions of nanopore-specific versus shared SVs. This distinguishes novel long-read-only variants—largely retrotransposon insertions—from those detectable by short-read Illumina calling.

## When to use

Use when you have a long-read (e.g., MinION NanoSV) SV VCF plus matched short-read call sets and want to characterize which novel SVs overlap repeats or retrotransposon insertions.

## When NOT to use

- The input call set is already intersected with short-read calls, so a novelty comparison would double-count shared variants
- The sequencing data are not human patient genomes with matched short-read data from the same individuals, since the shared-vs-specific classification requires both call sets on the same reference genome
- The goal is de novo complex rearrangement (e.g., chromothripsis) breakpoint resolution, which requires read-based phasing rather than repeat-content annotation

## Inputs

- NanoSV VCF call sets for Patient 1 and Patient 2 (long-read MinION, 11-16x coverage)
- SV call outputs from six Illumina callers: Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller
- 1KG (1000 Genomes Project) variant sites for novelty filtering
- UCSC RepeatMasker annotations
- DFAM annotations

## Outputs

- nanopore_sv_comparison_summary.csv with per-class SV counts, size bins, repeat/LINE fractions, and mean GC content per category
- Classification of each nanopore SV as shared or nanopore-specific (novel)
- List of SVs flagged as retrotransposon/LINE insertions

## How to apply

Start from the NanoSV VCF call sets for Patient 1 and Patient 2 and intersect each SV against the SV outputs of six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) using breakpoint-proximity/reciprocal-overlap criteria, removing variants present at 1KG variant sites to define the nanopore-specific (novel) set. Annotate repeat content of the surviving SVs using UCSC RepeatMasker and DFAM annotations to flag retrotransposon/LINE insertions. Compute SV class distribution, size distribution of nanopore-specific deletions (focusing on calls <200 bp), and GC content at nanopore-specific versus shared SV sites. Aggregate the contrasts into a summary table (per-class counts, size bins, repeat/LINE fractions, mean GC per category) written to nanopore_sv_comparison_summary.csv. The rationale is that genome-wide surveillance of inherited SVs from nanopore reads reveals novel variants missed in short-read data sets, a large proportion of which are retrotransposon insertions.

## Related tools

- **NanoSV** (Produces the long-read SV VCF call sets that are then classified as shared or nanopore-specific and annotated for repeat content) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts and sorts LAST-generated SAM alignments into coordinate-sorted BAM files used as NanoSV input) — https://github.com/biod/sambamba
- **Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)** (Provide the short-read SV call sets against which nanopore calls are intersected to define the novel set)

## Examples

```
NanoSV -s sambamba -b human_hg19.bed -o patient1_sv.vcf reads.sorted.bam
```

## Evaluation signals

- The computed fraction of nanopore-specific SVs falls near the reported ~14% novel fraction (944 SVs) and is well below the 89% reported for PB data
- The SV class distribution is deletion-dominated
- Nanopore-specific deletions are enriched below 200 bp in size
- Nanopore-specific SV sites show elevated LINE content and higher mean GC content than shared SV sites
- Novelty filtering removes SVs matching 1KG variant sites and those intersecting all six Illumina call sets under breakpoint-proximity/reciprocal-overlap criteria

## Limitations

- Extremely stringent post-calling filtering reduces estimated sensitivity to ~75%, and conservative SV calling and post-calling filtering steps lower the fraction of novel variants
- Due to medium coverage (11-16x) and intrinsic nanopore sequencing biases, some SVs may be missed
- Creating larger catalogs of SVs in complex repeat regions and segmental duplications is a particular challenge

## Evidence

- [abstract] novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
- [discussion] this percentage of novel variants is lower than for previously reported PB data (89%): "this percentage of novel variants is lower than for previously reported PB data (89%)"
- [discussion] Creating larger catalogs of SVs, in complex repeat regions and segmental duplications, is a particular challenge in the coming years: "Creating larger catalogs of SVs, in complex repeat regions and segmental duplications, is a particular challenge in the coming years"
- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data, such as data produced by Oxford Nanopore Technologies"
- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
