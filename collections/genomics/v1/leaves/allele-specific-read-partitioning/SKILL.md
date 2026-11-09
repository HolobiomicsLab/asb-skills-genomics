---
name: allele-specific-read-partitioning
description: Use when ONT reads (e.g., HG002 R10.4 BAM) must be partitioned into haplotype-specific
  subsets (HP:i:1 / HP:i:2) to enable allele-specific methylation calling downstream.
when_to_use_negative:
- No phased heterozygous variant VCF is available for the sample, so whatshap haplotag
  cannot assign informative tags to reads
- The analysis goal is per-site methylation calling only, with no interest in allele-specific
  methylation patterns or parent-of-origin partitioning
- Input is short-read data lacking overlaps with informative heterozygous variants;
  HP tagging would yield mostly unassigned reads
tools:
- name: whatshap
  role: haplotag tool that annotates each nanopore read with HP:i:1 or HP:i:2 based
    on overlapping heterozygous variants from the phased VCF
- name: Oxford Nanopore Technology (ONT)
  role: source platform for the single-molecule nanopore reads being partitioned
- name: DeepBAM
  role: downstream CpG methylation caller applied to haplotype-partitioned reads to
    unravel allele-specific methylation patterns
  repo: github.com/xiaochuanle/DeepBAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:0146eaa51be7173bbc84e0857b608f19f8bbf1a5106549c8e62fab98fb0d1b4f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - Oxford Nanopore Technology (ONT)
  - whatshap
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# allele-specific-read-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns Oxford Nanopore reads to paternal or maternal haplotypes by tagging reads with whatshap using a phased VCF, then splitting the tagged BAM into hp1/hp2 read sets to enable allele-specific methylation analysis as done in the DeepBAM study.

## When to use

Use when ONT reads (e.g., HG002 R10.4 BAM) must be partitioned into haplotype-specific subsets (HP:i:1 / HP:i:2) to enable allele-specific methylation calling downstream.

## When NOT to use

- No phased heterozygous variant VCF is available for the sample, so whatshap haplotag cannot assign informative tags to reads
- The analysis goal is per-site methylation calling only, with no interest in allele-specific methylation patterns or parent-of-origin partitioning
- Input is short-read data lacking overlaps with informative heterozygous variants; HP tagging would yield mostly unassigned reads

## Inputs

- HG002 Oxford Nanopore R10.4 alignment BAM (mapped reads)
- Genome In a Bottle (GIAB) whole-genome phased variant VCF
- Reference genome for BAM/VCF coordinate consistency

## Outputs

- hp1.bam containing only reads tagged HP:i:1 (paternal)
- hp2.bam containing only reads tagged HP:i:2 (maternal)
- Sorted and indexed haplotype-tagged BAM
- Haplotype-assignment rate (HP-tagged reads / total aligned reads, expected ~0.80)

## How to apply

Load the HG002 Oxford Nanopore R10.4 alignment BAM together with the Genome In a Bottle (GIAB) whole-genome phased variant VCF. Run whatshap haplotag with the phased VCF so that each read is annotated with a haplotype tag (HP:i:1 or HP:i:2) based on the informative heterozygous variants it overlaps. Sort and index the haplotype-tagged BAM to enable fast subset extraction, then split it into hp1.bam and hp2.bam keeping only reads carrying HP:i:1 or HP:i:2 respectively. Compute the haplotype-assignment rate as the number of HP-tagged reads divided by the total number of aligned reads; validate it against the ~0.80 rate reported for HG002 R10.4 data before feeding the read sets to methylation calling.

## Related tools

- **whatshap** (haplotag tool that annotates each nanopore read with HP:i:1 or HP:i:2 based on overlapping heterozygous variants from the phased VCF)
- **Oxford Nanopore Technology (ONT)** (source platform for the single-molecule nanopore reads being partitioned)
- **DeepBAM** (downstream CpG methylation caller applied to haplotype-partitioned reads to unravel allele-specific methylation patterns) — github.com/xiaochuanle/DeepBAM

## Examples

```
whatshap haplotag -o haplotagged.bam --reference ref.fa GIAB_phased.vcf.gz HG002_R10.4.bam; samtools sort -o haplotagged.sorted.bam haplotagged.bam; samtools index haplotagged.sorted.bam; samtools view -h haplotagged.sorted.bam | grep -v '^@' | awk '$0 ~ /HP:i:1/' > hp1.sam
```

## Evaluation signals

- Haplotype-assignment rate (HP-tagged reads / total aligned reads) is approximately 0.80, matching the ~80% rate reported for HG002 R10.4 data
- hp1.bam and hp2.bam both exist and are non-empty after splitting
- Every read in hp1.bam carries HP:i:1 and every read in hp2.bam carries HP:i:2, with no cross-contamination of tags
- The haplotype-tagged BAM is coordinate-sorted and indexed so subset extraction is fast and valid
- HP tags derive from reads overlapping informative heterozygous variants in the GIAB phased VCF

## Limitations

- Reads not overlapping informative heterozygous variants remain untagged, capping the assignment rate (~20% of HG002 R10.4 reads unassigned)
- Partition quality depends on the accuracy and density of the phased GIAB variant VCF; phasing errors propagate into misassigned reads
- Allele-specific interpretation assumes the GIAB haplotype labels (hp1/hp2) correspond to the desired parental origin of the sample

## Evidence

- [methods] Haplotype assignment was performed by tagging reads with whatshap using phased VCF files against mapped BAMs, then extracting reads corresponding to hp1 (HP:i:1) and hp2 (HP:i:2).: [evidence span withheld — anchor status: not_found]
- [methods] confirm the haplotype-assignment rate is approximately 0.80 (the ~80% assignment rate reported for HG002 R10.4 data): [evidence span withheld — anchor status: not_found]
- [abstract] It enables unraveling allele-specific methylation patterns, including regions of transposable elements: "It enables unraveling allele-specific methylation patterns, including regions of transposable elements"
- [intro] Single-molecule sequencing techniques, such as those used on the Oxford Nanopore Technology (ONT) platform: "Single-molecule sequencing techniques, such as those used on the Oxford Nanopore Technology (ONT) platform"
