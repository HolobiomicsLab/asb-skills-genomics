---
name: structural-variant-phasing-accuracy-benchmarking
description: Use when long nanopore reads must be assigned to parental haplotypes
  via trio-phased heterozygous SNP anchors, or when SV calls need phasing from haplotype-assigned
  flanking reads and the result must be validated against trio-phased SNP ground truth.
when_to_use_negative:
- Input is short-read Illumina data only — the paper's phasing depends on reads long
  enough to span multiple heterozygous SNPs and SV breakpoints; Illumina data even
  missed many of these SVs entirely.
- No trio (or other externally phased) heterozygous SNP ground truth exists for the
  sample — the benchmark and the haplotype anchoring both require trio-phased SNPs.
- Goal is de novo genome assembly from the nanopore reads — the authors did not attempt
  this 'primarily due to insufficient coverage' (11–16×).
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LAST
  role: Aligns MinION reads to the reference; its alignments are the input for NanoSV
    SV calling and for read-level haplotype assignment
  repo: https://github.com/mroosmalen/nanosv
- name: NanoSV
  role: Produces the long-read SV call set whose heterozygous SVs are phased and benchmarked
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts LAST MAF output to SAM/BAM and coordinate-sorts it to prepare the
    phased-read input
  repo: https://github.com/biod/sambamba
- name: GATK PBT
  role: Supplies the trio-phased heterozygous SNP ground truth used both as haplotype
    anchors and as the phasing-accuracy benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:e2da8c10535a7878eb14dab8697b0caaa7f6d25dc3c869226a267cc6f6695fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - GATK PBT
  - LAST
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# structural-variant-phasing-accuracy-benchmarking

## Summary

Phases MinION nanopore reads against trio-phased (GATK PBT) heterozygous SNPs by read-level allele log-likelihood, then uses haplotype-assigned reads flanking SV breakpoints to phase Manta/NanoSV calls and benchmark phasing accuracy against the trio ground truth. This is the mechanism the paper used to show all 40 de novo chromothripsis breakpoint junctions in Patient 1 are paternal and to achieve 1909/2389 SVs phased at 98.5% concordance.

## When to use

Use when long nanopore reads must be assigned to parental haplotypes via trio-phased heterozygous SNP anchors, or when SV calls need phasing from haplotype-assigned flanking reads and the result must be validated against trio-phased SNP ground truth.

## When NOT to use

- Input is short-read Illumina data only — the paper's phasing depends on reads long enough to span multiple heterozygous SNPs and SV breakpoints; Illumina data even missed many of these SVs entirely.
- No trio (or other externally phased) heterozygous SNP ground truth exists for the sample — the benchmark and the haplotype anchoring both require trio-phased SNPs.
- Goal is de novo genome assembly from the nanopore reads — the authors did not attempt this 'primarily due to insufficient coverage' (11–16×).

## Inputs

- Coordinate-sorted and indexed BAM of MinION nanopore reads aligned with LAST (MAF converted to SAM/BAM via maf-convert and sambamba)
- GATK pedigree-based trio (PBT) list of heterozygous SNPs with parental haplotype phase (ground truth)
- Structural variant call set (NanoSV, plus Manta/NanoSV-overlapping heterozygous SVs) with breakpoint coordinates

## Outputs

- Per-read paternal/maternal haplotype assignments
- Parental-origin assignment for each de novo breakpoint junction (all 40 junctions assigned paternal in Patient 1)
- Per-SV phasing assignments and concordance metrics table (sv_phasing_benchmark.tsv): 1909/2389 SVs phased at 98.5% concordance

## How to apply

Start from the LAST-aligned MinION BAM (LAST alignment was chosen because it 'gives the most accurate results for SV calling with NanoSV') and the GATK pedigree-based trio (PBT) heterozygous SNP list serving as ground-truth haplotype anchors. For each nanopore read overlapping a trio-phased heterozygous SNP, sum the SNP allele log-likelihoods — incorporating base qualities and mapping qualities — and assign the read to the paternal or maternal haplotype by the read-likelihood method. Partition breakpoint-junction-supporting reads and reference reads by assigned haplotype to determine parental origin of junctions (yielding the paternal origin of all 40 de novo chromothripsis junctions, Fig. 2). For each heterozygous SV overlapping the Manta/NanoSV call sets (2389 SVs in total), phase the SV by comparing haplotypes of breakpoint-flanking haplotype-assigned reads. Finally, compare each phased SV against the GATK PBT ground truth and write per-SV assignments plus concordance metrics (e.g. to sv_phasing_benchmark.tsv).

## Related tools

- **LAST** (Aligns MinION reads to the reference; its alignments are the input for NanoSV SV calling and for read-level haplotype assignment) — https://github.com/mroosmalen/nanosv
- **NanoSV** (Produces the long-read SV call set whose heterozygous SVs are phased and benchmarked) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts LAST MAF output to SAM/BAM and coordinate-sorts it to prepare the phased-read input) — https://github.com/biod/sambamba
- **GATK PBT** (Supplies the trio-phased heterozygous SNP ground truth used both as haplotype anchors and as the phasing-accuracy benchmark)

## Examples

```
lastal -Q1 -p my-params referencedb reads.fastq | last-split | maf-convert -f reference.dict sam /dev/stdin | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam
```

## Evaluation signals

- Reproduce 1909/2389 heterozygous SVs successfully phased against the Manta/NanoSV-overlapping call set
- Phasing concordance with GATK PBT trio-phased heterozygous SNP ground truth must reach 98.5%
- All 40 de novo chromothripsis breakpoint junctions in Patient 1 must be assigned paternal origin (Fig. 2 reproduction)
- Per-SV output table (sv_phasing_benchmark.tsv) contains a phasing assignment and concordance metric for each SV
- Upstream SV calls should match the paper's precision benchmark (>95% precision) before phasing is attempted

## Limitations

- Phasing coverage is limited: SV pipeline sensitivity was ~75% due to 'extremely stringent filtering', medium coverage (11–16×), and intrinsic nanopore sequencing biases
- Only ~80% of SVs (1909/2389) could be phased — SVs lacking haplotype-assigned reads at breakpoints remain unphased
- Simultaneous SNV calling with phasing was deferred by the authors pending MinION/ONT data quality and throughput increases
- LAST mapping is compute-intensive; the README notes 'mapping with LAST requires more compute resources'

## Evidence

- [discussion] Read-based phasing of SVs using flanking phased heterozygous SNPs: [evidence span withheld — anchor status: not_found]
- [discussion] Haplotype read separation for chromothripsis reconstruction: "we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements"
- [readme] LAST alignment preferred for NanoSV input: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] NanoSV breakpoint-junction algorithm: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
