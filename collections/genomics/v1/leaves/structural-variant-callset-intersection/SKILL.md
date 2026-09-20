---
name: structural-variant-callset-intersection
description: Use when you have a long-read (nanopore) SV VCF plus matched short-read
  SV callsets for the same genomes and need to quantify which SVs are missed by short
  reads or absent from 1KG, including their size/GC/repeat properties.
when_to_use_negative:
- Calling de novo chromothripsis rearrangement breakpoints or phasing them by haplotype
  - that is a different task in this paper, not callset intersection
- Only one callset (no matched short-read or population data) exists, so shared-versus-novel
  classification cannot be computed
- Attempting a full de novo human genome assembly from MinION reads - the authors
  did not attempt this due to insufficient coverage
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanoSV
  role: Produces the query nanopore SV VCF callset from split- and gapped-aligned
    long-read BAMs; these calls are then intersected against short-read callsets
  repo: mroosmalen/nanosv
- name: LAST
  role: Long-read mapper preferred for NanoSV input; its alignments define the breakpoint-junctions
    that populate the nanopore callset being compared
- name: sambamba
  role: Converts LAST MAF output to sorted BAM (view -h -S --format=bam, sort) so
    coordinate-sorted, indexed BAMs feed NanoSV
  repo: biod/sambamba
- name: Pindel
  role: One of the six Illumina short-read SV/variant callers whose callsets define
    the comparison baseline
- name: Manta
  role: One of the six Illumina short-read SV callers intersected against the nanopore
    callset
- name: Delly
  role: One of the six Illumina short-read SV callers intersected against the nanopore
    callset
- name: FREEC
  role: One of the six Illumina short-read callers whose calls define the shared-SV
    baseline
- name: Mobster
  role: One of the six Illumina short-read callers whose calls define the shared-SV
    baseline
- name: GATK HaplotypeCaller
  role: One of the six Illumina variant callers whose calls define the shared-SV baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:614d25c744bfea7d5d9aedd1259b7dc8224a57118ca0c871163d5310279bad6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FREEC
  - GATK HaplotypeCaller
  - LAST
  - Manta
  - Mobster
  - NanoSV
  - Pindel
  - sambamba
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# structural-variant-callset-intersection

## Summary

Intersects NanoSV structural-variant callsets from MinION nanopore sequencing of patient genomes against SV calls from six Illumina short-read callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) and 1KG variant sites to define nanopore-specific (novel) SVs, then profiles their class, size, GC, and repeat/LINE composition.

## When to use

Use when you have a long-read (nanopore) SV VCF plus matched short-read SV callsets for the same genomes and need to quantify which SVs are missed by short reads or absent from 1KG, including their size/GC/repeat properties.

## When NOT to use

- Calling de novo chromothripsis rearrangement breakpoints or phasing them by haplotype - that is a different task in this paper, not callset intersection
- Only one callset (no matched short-read or population data) exists, so shared-versus-novel classification cannot be computed
- Attempting a full de novo human genome assembly from MinION reads - the authors did not attempt this due to insufficient coverage

## Inputs

- NanoSV VCF callsets for nanopore-sequenced Patient 1 and Patient 2 genomes (11-16x MinION coverage)
- SV call VCFs from six Illumina short-read callers: Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller
- 1KG (1000 Genomes Project) variant sites for novelty filtering
- UCSC RepeatMasker and DFAM repeat annotations
- Coordinate-sorted, indexed long-read BAM files (LAST-mapped MinION reads)

## Outputs

- Per-SV classification table labeling each nanopore SV as shared, Illumina-detectable, 1KG-matched, or nanopore-specific
- nanopore_sv_comparison_summary.csv with per-SV-class counts, deletion size bins, repeat/LINE fractions, and mean GC content per category
- Novel-variant fraction statistic (expected near ~14% of the combined Patient 1/2 nanopore callset)

## How to apply

Load the NanoSV VCFs for Patient 1 and Patient 2 (produced from LAST-mapped MinION reads at 11-16x depth) together with SV call outputs from the six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) built on the same patient genomes. Classify each nanopore SV as shared or nanopore-specific by intersecting against all six Illumina callsets using breakpoint-proximity or reciprocal-overlap criteria, then remove SVs matching 1KG variant sites to define the novel set. Annotate repeat content with UCSC RepeatMasker and DFAM annotations to flag retrotransposon/LINE insertions. Compute the SV class distribution (expect deletion dominance), size bins for nanopore-specific deletions (focusing on calls <200 bp), and mean GC content at nanopore-specific versus shared sites. Aggregate into a summary table (nanopore_sv_comparison_summary.csv) with per-class counts, size bins, LINE fractions, and mean GC per category; expect the novel fraction to sit near the reported ~14%, far below the 89% reported for Pacific Biosciences data, partly because the paper's conservative SV calling and extremely stringent post-calling filtering lower the apparent novelty rate.

## Related tools

- **NanoSV** (Produces the query nanopore SV VCF callset from split- and gapped-aligned long-read BAMs; these calls are then intersected against short-read callsets) — mroosmalen/nanosv
- **LAST** (Long-read mapper preferred for NanoSV input; its alignments define the breakpoint-junctions that populate the nanopore callset being compared)
- **sambamba** (Converts LAST MAF output to sorted BAM (view -h -S --format=bam, sort) so coordinate-sorted, indexed BAMs feed NanoSV) — biod/sambamba
- **Pindel** (One of the six Illumina short-read SV/variant callers whose callsets define the comparison baseline)
- **Manta** (One of the six Illumina short-read SV callers intersected against the nanopore callset)
- **Delly** (One of the six Illumina short-read SV callers intersected against the nanopore callset)
- **FREEC** (One of the six Illumina short-read callers whose calls define the shared-SV baseline)
- **Mobster** (One of the six Illumina short-read callers whose calls define the shared-SV baseline)
- **GATK HaplotypeCaller** (One of the six Illumina variant callers whose calls define the shared-SV baseline)

## Examples

```
NanoSV -t 4 -s sambamba -b human_hg19.bed -o patient1_nanosv.vcf reads.sorted.bam
```

## Evaluation signals

- Fraction of SVs classified as nanopore-specific falls near the reported ~14% (944 SVs) of the combined Patient 1/2 callset, well below the 89% reported for Pacific Biosciences data
- SV class distribution of the novel set is deletion-dominated
- Nanopore-specific deletions are enriched in the <200 bp size bin relative to shared deletions
- Novel SV sites show elevated LINE/retrotransposon content in RepeatMasker/DFAM annotations versus shared SVs
- Mean GC content at nanopore-specific sites is higher than at shared SV sites, and the summary CSV contains per-class counts, size bins, LINE fractions, and GC columns

## Limitations

- The ~14% novel fraction is depressed by conservative SV calling and extremely stringent post-calling filtering, which also reduces estimated sensitivity to ~75%
- Medium coverage (11-16x) and intrinsic nanopore sequencing biases limit which SVs are recovered, so 'nanopore-specific' may partly reflect sensitivity differences rather than true absence in short-read data
- Creating larger SV catalogs in complex repeat regions and segmental duplications remains an unresolved challenge, affecting repeat-annotation interpretation

## Evidence

- [discussion] Six-caller Illumina comparison baseline: "could not be found in matching Illumina sequencing data (despite extensive variant calling using six different variant calling methods"
- [discussion] Expected novelty fraction vs PB benchmark: "this percentage of novel variants is lower than for previously reported PB data (89%)"
- [discussion] Filtering stringency lowers sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [abstract] Novel variants enriched for retrotransposons: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
- [readme] NanoSV scope on long reads: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data, such as data produced by Oxford Nanopore Technologies"
- [readme] LAST is the preferred mapper for NanoSV: "We found that LAST alignments give the most accurate results for SV calling with NanoSV."
