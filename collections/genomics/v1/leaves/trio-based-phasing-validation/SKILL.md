---
name: trio-based-phasing-validation
description: Use when long nanopore reads must be assigned to parental haplotypes
  using trio-phased heterozygous SNP anchors, to determine parental origin of SV/breakpoint
  junctions and benchmark SV phasing accuracy.
when_to_use_negative:
- Input reads are short Illumina reads — the paper's method requires long nanopore
  reads spanning flanking phased heterozygous SNPs; Illumina data missed the de novo
  chromothripsis rearrangements even with six variant calling methods
- No trio/pedigree sample is available — without GATK PBT trio-phased SNPs there are
  no ground-truth haplotype anchors for read assignment or benchmarking
- Coverage is far below the ~11–16x used here, so few reads overlap enough trio-phased
  SNPs for reliable haplotype likelihood assignment
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: LAST
  role: Long-read mapper producing the BAM alignments whose per-read SNP alleles are
    scored for haplotype likelihoods; the paper found LAST alignments give the most
    accurate NanoSV input
  repo: http://last.cbrc.jp/
- name: NanoSV
  role: SV caller producing the breakpoint-junction call set (2389 heterozygous SVs)
    that is subsequently phased against trio haplotypes
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: SAM-to-BAM conversion, sorting and indexing of the LAST MAF-derived alignments
    prior to phasing and SV calling
  repo: https://github.com/biod/sambamba
- name: MinION nanopore sequencer
  role: Source of the long nanopore reads (11–16x diploid coverage) that enable read-based
    phasing
- name: GATK PBT
  role: Pedigree-based trio phasing providing the ground-truth heterozygous SNP haplotype
    anchors and the benchmark for phasing concordance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:1b2da4893629a271b347c7f5de1975f5ba85bf4290b548a09102aa04390bbda7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GATK PBT
  - LAST
  - MinION nanopore sequencer
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# trio-based-phasing-validation

## Summary

Phase nanopore reads and structural variants against pedigree-based trio-phased heterozygous SNPs (GATK PBT), assigning each MinION read to the paternal or maternal haplotype by summed SNP allele log-likelihoods, then benchmarking per-SV phasing calls against the trio ground truth.

## When to use

Use when long nanopore reads must be assigned to parental haplotypes using trio-phased heterozygous SNP anchors, to determine parental origin of SV/breakpoint junctions and benchmark SV phasing accuracy.

## When NOT to use

- Input reads are short Illumina reads — the paper's method requires long nanopore reads spanning flanking phased heterozygous SNPs; Illumina data missed the de novo chromothripsis rearrangements even with six variant calling methods
- No trio/pedigree sample is available — without GATK PBT trio-phased SNPs there are no ground-truth haplotype anchors for read assignment or benchmarking
- Coverage is far below the ~11–16x used here, so few reads overlap enough trio-phased SNPs for reliable haplotype likelihood assignment

## Inputs

- LAST-aligned MinION nanopore read alignment (coordinate-sorted, indexed BAM), e.g. Patient 1 at ~16x coverage
- GATK pedigree-based trio (PBT) phased heterozygous SNP list (VCF) used as ground-truth haplotype anchors
- Pedigree/trio relationship file defining paternal and maternal haplotypes
- SV call set from NanoSV (and/or Manta) with breakpoint-junction positions, e.g. 2389 heterozygous SVs

## Outputs

- Per-read haplotype assignments (paternal vs maternal) with SNP allele log-likelihood scores
- Parental-origin assignment for each de novo breakpoint junction (40 junctions, all paternal in Patient 1, Fig. 2)
- Per-SV phasing assignments and concordance metrics written to sv_phasing_benchmark.tsv

## How to apply

Load the LAST-aligned MinION read BAM (Patient 1 at 11–16x coverage) together with the GATK pedigree-based trio (PBT) heterozygous SNP list used as ground-truth haplotype anchors. For each read overlapping a trio-phased heterozygous SNP, sum SNP allele log-likelihoods incorporating base quality and mapping quality to assign the read to the paternal or maternal haplotype. Partition breakpoint-junction supporting reads and reference reads by assigned haplotype to call parental origin (all 40 de novo chromothripsis junctions in Patient 1 were paternal). For each of the 2389 Manta/NanoSV-overlapping heterozygous SVs, phase the SV by comparing the haplotypes of breakpoint-flanking haplotype-assigned reads. Evaluate against the GATK PBT ground truth: the paper recovered 1909/2389 phased SVs at 98.5% concordance, the acceptance criterion for this workflow.

## Related tools

- **LAST** (Long-read mapper producing the BAM alignments whose per-read SNP alleles are scored for haplotype likelihoods; the paper found LAST alignments give the most accurate NanoSV input) — http://last.cbrc.jp/
- **NanoSV** (SV caller producing the breakpoint-junction call set (2389 heterozygous SVs) that is subsequently phased against trio haplotypes) — https://github.com/mroosmalen/nanosv
- **sambamba** (SAM-to-BAM conversion, sorting and indexing of the LAST MAF-derived alignments prior to phasing and SV calling) — https://github.com/biod/sambamba
- **MinION nanopore sequencer** (Source of the long nanopore reads (11–16x diploid coverage) that enable read-based phasing)
- **GATK PBT** (Pedigree-based trio phasing providing the ground-truth heterozygous SNP haplotype anchors and the benchmark for phasing concordance)

## Examples

```
NanoSV -t 8 -s sambamba -o nanosv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- Benchmarking against GATK PBT ground-truth heterozygous SNPs reproduces 1909/2389 SVs phased at 98.5% phasing concordance
- All 40 de novo chromothripsis breakpoint junctions in Patient 1 are assigned paternal origin (Fig. 2 reproduction)
- Input BAM is coordinate-sorted and indexed (required by NanoSV/phasing scripts)
- Upstream NanoSV calls used config defaults such as min_mapq = 20, min_pid = 0.7 and cluster_count = 2, so low-quality segments are excluded before phasing
- sv_phasing_benchmark.tsv contains per-SV phasing assignment plus a concordance metric for every overlapping heterozygous SV

## Limitations

- Only 1909 of 2389 heterozygous SVs could be phased (~80%) at the 11–16x nanopore coverage used; the remainder lacked sufficient overlapping haplotype-assigned reads
- Sensitivity is limited by medium coverage and intrinsic nanopore sequencing biases, and extremely stringent post-calling filtering reduced estimated sensitivity to ~75%
- Simultaneous SNV calling with phasing is conditional on future MinION/ONT data quality and throughput increases; SNV calls in Patient 1 were not further investigated
- NanoSV was not attempted on a full de novo human genome assembly from these reads due to insufficient coverage

## Evidence

- [discussion] we here devised a computational strategy that allowed accurate phasing of SVs directly from the long nanopore reads using flanking (phased) heterozygous SNPs: [evidence span withheld — anchor status: not_found]
- [abstract] we leveraged to determine the parental origin of all de novo chromothripsis breakpoints and to resolve the structure of these complex rearrangements: "we leveraged to determine the parental origin of all de novo chromothripsis breakpoints and to resolve the structure of these complex rearrangements"
- [discussion] we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements: "we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements"
- [readme] The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions"
- [discussion] We were able to extract all known de novo breakpoint junctions for Patient 1 (Fig. 1), even at relatively low coverage (16×): "We were able to extract all known de novo breakpoint junctions for Patient 1 (Fig. 1), even at relatively low coverage (16×)"
