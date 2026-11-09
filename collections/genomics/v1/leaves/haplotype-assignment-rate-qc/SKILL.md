---
name: haplotype-assignment-rate-qc
description: Use when nanopore reads (e.g. HG002 Oxford Nanopore R10.4) have been
  haplotagged with whatshap against a GIAB phased VCF and you must validate HP:i:1/HP:i:2
  assignment quality before allele-specific methylation analysis.
when_to_use_negative:
- No phased VCF is available for the sample or the organism lacks informative heterozygous
  variants — haplotag cannot assign HP tags and the QC rate is meaningless
- The goal is CpG methylation calling itself (e.g. running DeepBAM extract_and_call_mods)
  rather than validating haplotype separation prior to allele-specific analysis
- Input reads are not long nanopore reads or are already consistently HP-tagged and
  split, so re-tagging and re-computing the rate adds no information
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: whatshap
  role: Annotates each nanopore read with a haplotype tag (HP:i:1 or HP:i:2) via `whatshap
    haplotag` using the phased VCF against the mapped BAM; the tagged reads are then
    split and the assignment rate computed
- name: Oxford Nanopore Technology (ONT)
  role: Source platform of the R10.4 long-read data whose reads are haplotagged and
    whose ~80% assignment rate serves as the QC benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:6e28cf9c03d693af69af22b49e8defd5b1e5538d1c022f866a785e837f64f57b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - Oxford Nanopore Technology (ONT)
  - whatshap
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# haplotype-assignment-rate-qc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

QC skill for whatshap-based haplotype assignment of nanopore reads: compute the fraction of HP-tagged reads and verify the hp1/hp2 BAM split so that allele-specific methylation analysis (as in DeepBAM on HG002 R10.4 data) rests on correctly assigned haplotypes.

## When to use

Use when nanopore reads (e.g. HG002 Oxford Nanopore R10.4) have been haplotagged with whatshap against a GIAB phased VCF and you must validate HP:i:1/HP:i:2 assignment quality before allele-specific methylation analysis.

## When NOT to use

- No phased VCF is available for the sample or the organism lacks informative heterozygous variants — haplotag cannot assign HP tags and the QC rate is meaningless
- The goal is CpG methylation calling itself (e.g. running DeepBAM extract_and_call_mods) rather than validating haplotype separation prior to allele-specific analysis
- Input reads are not long nanopore reads or are already consistently HP-tagged and split, so re-tagging and re-computing the rate adds no information

## Inputs

- Haplotype-tagged alignment BAM produced by whatshap haplotag (reads carrying HP:i:1 or HP:i:2 tags)
- GIAB whole-genome phased variant VCF used for haplotagging
- Mapped HG002 Oxford Nanopore R10.4 alignment BAM (for total aligned-read denominator)

## Outputs

- hp1.bam containing only reads tagged HP:i:1
- hp2.bam containing only reads tagged HP:i:2
- Haplotype-assignment rate (HP-tagged reads / total aligned reads), expected ~0.80 for HG002 R10.4 data

## How to apply

Tag the mapped HG002 Oxford Nanopore R10.4 alignment BAM with whatshap haplotag using the Genome In a Bottle (GIAB) whole-genome phased variant VCF, so each read overlapping informative heterozygous variants receives an HP:i:1 or HP:i:2 tag. Sort and index the haplotype-tagged BAM to enable fast subset extraction, then split it into hp1.bam and hp2.bam keeping only reads carrying HP:i:1 and HP:i:2 respectively. Compute the haplotype-assignment rate as the number of HP-tagged reads divided by the total number of aligned reads; for HG002 R10.4 data this rate should be approximately 0.80, matching the ~80% assignment reported for this dataset. Finally, verify that hp1.bam and hp2.bam exist and contain only consistently HP-tagged reads before proceeding to allele-specific methylation calling.

## Related tools

- **whatshap** (Annotates each nanopore read with a haplotype tag (HP:i:1 or HP:i:2) via `whatshap haplotag` using the phased VCF against the mapped BAM; the tagged reads are then split and the assignment rate computed)
- **Oxford Nanopore Technology (ONT)** (Source platform of the R10.4 long-read data whose reads are haplotagged and whose ~80% assignment rate serves as the QC benchmark)

## Examples

```
whatshap haplotag -o hg002.haplotag.bam --reference ref.fa giab.phased.vcf.gz hg002.r104.bam; samtools view -b -d HP:Y:1 hg002.haplotag.bam > hp1.bam
```

## Evaluation signals

- Haplotype-assignment rate (HP-tagged reads / total aligned reads) is approximately 0.80 for HG002 R10.4 data
- hp1.bam and hp2.bam both exist and are non-empty
- Every read in hp1.bam carries HP:i:1 and every read in hp2.bam carries HP:i:2 (no cross-contaminated tags)
- The haplotype-tagged BAM is sorted and indexed before subset extraction, so the split reads can be pulled out quickly

## Limitations

- About 20% of aligned reads remain unassigned (rate ~0.80), so allele-specific analyses will have reduced coverage per haplotype
- Assignment depends on reads overlapping informative heterozygous variants in the phased VCF; reads in low-heterozygosity regions cannot be tagged
- The ~0.80 benchmark was established for HG002 R10.4 data and may not transfer to other samples, chemistries, or read sets

## Evidence

- [other] Haplotype assignment was performed by tagging reads with whatshap using phased VCF files against mapped BAMs, then extracting reads corresponding to hp1 (HP:i:1) and hp2 (HP:i:2).: [evidence span withheld — anchor status: not_found]
- [other] Compute the haplotype-assignment rate as the number of HP-tagged reads divided by the total number of aligned reads.: [evidence span withheld — anchor status: not_found]
- [other] confirm the haplotype-assignment rate is approximately 0.80 (the ~80% assignment rate reported for HG002 R10.4 data): [evidence span withheld — anchor status: not_found]
