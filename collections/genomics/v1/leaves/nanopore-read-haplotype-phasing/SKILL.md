---
name: nanopore-read-haplotype-phasing
description: Use when long nanopore reads must be assigned to a parental haplotype
  using trio-phased heterozygous SNPs, e.g. to phase SVs or determine parental origin
  of de novo breakpoint junctions.
when_to_use_negative:
- When only short-read (Illumina) data are available — the method requires long MinION
  reads spanning multiple heterozygous SNPs to assign haplotype per read
- When no trio-phased (GATK PBT) heterozygous SNP ground truth exists to anchor the
  haplotypes
- When the goal is de novo genome assembly rather than read-level phasing — the authors
  did not attempt assembly due to insufficient coverage (11–16×)
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: LAST
  role: Aligns MinION reads to the reference genome; LAST alignments were found to
    give the most accurate results for SV calling with NanoSV and provide the mapped
    reads used for phasing
- name: NanoSV
  role: Calls SVs from the long-read BAM, producing the breakpoint-junction reads
    and heterozygous SV set that are subsequently phased by haplotype
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts and sorts LAST MAF-derived SAM output into coordinate-sorted, indexed
    BAM for phasing and SV calling
  repo: https://github.com/biod/sambamba
- name: MinION nanopore sequencer
  role: Produces the long nanopore reads (11–16× coverage) whose SNP-overlapping bases
    supply the haplotype log-likelihood evidence
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:3630cb10e2047a6ff8c6563fd1075ac3d893be69a2c2c9838cf22cbdd7b5e3ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - MinION nanopore sequencer
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nanopore-read-haplotype-phasing

## Summary

Assign individual MinION nanopore reads to paternal or maternal haplotypes by summing SNP allele log-likelihoods (incorporating base and mapping qualities) over trio-phased (GATK PBT) heterozygous SNPs overlapped by each read, enabling read-based phasing of SVs and determination of the parental origin of de novo chromothripsis breakpoints.

## When to use

Use when long nanopore reads must be assigned to a parental haplotype using trio-phased heterozygous SNPs, e.g. to phase SVs or determine parental origin of de novo breakpoint junctions.

## When NOT to use

- When only short-read (Illumina) data are available — the method requires long MinION reads spanning multiple heterozygous SNPs to assign haplotype per read
- When no trio-phased (GATK PBT) heterozygous SNP ground truth exists to anchor the haplotypes
- When the goal is de novo genome assembly rather than read-level phasing — the authors did not attempt assembly due to insufficient coverage (11–16×)

## Inputs

- LAST-aligned Patient 1 MinION read alignment (coordinate-sorted, indexed BAM)
- GATK pedigree-based trio (PBT) phased heterozygous SNP list (ground-truth haplotype anchors)
- Patient pedigree/trio information (paternal and maternal haplotype labels)
- Chromothripsis breakpoint-junction supporting reads and reference reads (from NanoSV/Manta SV calling)

## Outputs

- Per-read paternal/maternal haplotype assignments for reads overlapping trio-phased heterozygous SNPs
- Per-SV parental phasing assignments for 2389 Manta/NanoSV-overlapping heterozygous SVs (sv_phasing_benchmark.tsv)
- Concordance metrics of phased SVs vs GATK PBT ground truth (1909/2389 phased, 98.5% accuracy)
- Parental-origin assignment of all 40 de novo chromothripsis breakpoint junctions (Fig. 2)

## How to apply

Load the LAST-aligned Patient 1 MinION read BAM together with the GATK pedigree-based trio (PBT) heterozygous SNP list used as ground-truth haplotype anchors. For each read overlapping a trio-phased heterozygous SNP, sum SNP allele log-likelihoods incorporating base and mapping qualities to assign the read to the paternal or maternal haplotype via the paper's read-likelihood method. Partition chromothripsis breakpoint-junction supporting reads and reference reads by assigned parental haplotype to reproduce the paternal-origin assignment of all 40 de novo junctions in Patient 1 (Fig. 2). For each of the 2389 Manta/NanoSV-overlapping heterozygous SVs, phase the SV by comparing the haplotypes of breakpoint-flanking haplotype-assigned reads. Evaluate phased SV calls against the GATK PBT ground truth, recovering 1909/2389 phased SVs at 98.5% phasing concordance, and write per-SV assignments and concordance metrics to sv_phasing_benchmark.tsv.

## Related tools

- **LAST** (Aligns MinION reads to the reference genome; LAST alignments were found to give the most accurate results for SV calling with NanoSV and provide the mapped reads used for phasing)
- **NanoSV** (Calls SVs from the long-read BAM, producing the breakpoint-junction reads and heterozygous SV set that are subsequently phased by haplotype) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts and sorts LAST MAF-derived SAM output into coordinate-sorted, indexed BAM for phasing and SV calling) — https://github.com/biod/sambamba
- **MinION nanopore sequencer** (Produces the long nanopore reads (11–16× coverage) whose SNP-overlapping bases supply the haplotype log-likelihood evidence)

## Examples

```
sambamba sort /dev/stdin -o reads.sorted.bam && NanoSV -s sambamba -o nanosv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- All 40 de novo chromothripsis breakpoint junctions in Patient 1 are assigned to the paternal haplotype, reproducing Fig. 2
- 1909 of 2389 heterozygous SVs overlapping Manta/NanoSV calls are successfully phased
- Phased SV concordance with GATK PBT ground truth reaches 98.5%
- Per-SV phasing output written to sv_phasing_benchmark.tsv contains per-SV parental assignments plus concordance metrics
- Per-read haplotype log-likelihood sums are computed only over reads overlapping trio-phased heterozygous SNPs, using base and mapping qualities

## Limitations

- Phasing sensitivity is limited by medium coverage (11–16×) and intrinsic nanopore sequencing biases
- Not all SVs can be phased — only 1909/2389 heterozygous SVs were phased; the rest lack sufficient haplotype-assigned flanking reads
- The paper used extremely stringent post-calling filtering that reduces estimated sensitivity to ~75%, affecting which SVs enter the phasing benchmark
- Simultaneous SNV calling with phasing was considered conditional on future MinION/ONT data quality and throughput increases

## Evidence

- [discussion] read-based phasing of SVs using flanking phased heterozygous SNPs: [evidence span withheld — anchor status: not_found]
- [discussion] read separation by haplotype for chromothripsis reconstruction: "we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements"
- [discussion] coverage limitations: "Due to the medium coverage, some intrinsic nanopore sequencing biases and for benchmarking purposes"
- [readme] LAST mapper choice: "We found that LAST alignments give the most accurate results for SV calling with NanoSV."
- [readme] NanoSV breakpoint clustering: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
