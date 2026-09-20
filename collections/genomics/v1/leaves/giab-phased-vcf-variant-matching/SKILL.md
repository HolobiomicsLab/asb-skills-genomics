---
name: giab-phased-vcf-variant-matching
description: Use when aligned ONT reads (e.g., HG002 R10.4 BAM) must be partitioned
  by parental haplotype to enable allele-specific methylation downstream, and a GIAB
  phased heterozygous variant VCF is available.
when_to_use_negative:
- No phased heterozygous variant VCF exists for the sample or the genome is effectively
  haploid, so informative heterozygous sites cannot partition reads.
- Input reads are not aligned (raw POD5/FASTQ) — reads must first be mapped and sorted
  into a BAM before haplotagging.
- The goal is single-molecule CpG methylation calling itself — that is performed by
  DeepBAM/Dorado on the reads, independent of haplotype assignment.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: whatshap
  role: Runs haplotag to annotate each aligned read with HP:i:1 or HP:i:2 based on
    informative heterozygous variants it overlaps, using the GIAB phased VCF
- name: Oxford Nanopore Technology (ONT)
  role: Source platform of the HG002 R10.4 nanopore reads being haplotype-tagged
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:b65cbd69978656972eb8cc21b082131854550095b9951f2fad06e4c62dcb809e
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# giab-phased-vcf-variant-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns individual ONT reads to paternal (HP:i:1) or maternal (HP:i:2) haplotypes by running whatshap haplotag with a GIAB phased variant VCF against the mapped BAM, then splitting tagged reads into hp1/hp2 read sets for allele-specific methylation analysis.

## When to use

Use when aligned ONT reads (e.g., HG002 R10.4 BAM) must be partitioned by parental haplotype to enable allele-specific methylation downstream, and a GIAB phased heterozygous variant VCF is available.

## When NOT to use

- No phased heterozygous variant VCF exists for the sample or the genome is effectively haploid, so informative heterozygous sites cannot partition reads.
- Input reads are not aligned (raw POD5/FASTQ) — reads must first be mapped and sorted into a BAM before haplotagging.
- The goal is single-molecule CpG methylation calling itself — that is performed by DeepBAM/Dorado on the reads, independent of haplotype assignment.

## Inputs

- Alignment BAM of HG002 Oxford Nanopore R10.4 reads mapped to the reference genome
- Genome In a Bottle (GIAB) whole-genome phased variant VCF
- Reference genome FASTA matching the VCF and BAM

## Outputs

- Haplotype-tagged BAM with HP:i:1 / HP:i:2 read tags
- hp1.bam (reads carrying HP:i:1)
- hp2.bam (reads carrying HP:i:2)
- Haplotype-assignment rate (number of HP-tagged reads / total aligned reads)

## How to apply

Load the HG002 Oxford Nanopore R10.4 alignment BAM and the Genome In a Bottle (GIAB) whole-genome phased variant VCF. Run whatshap haplotag with the phased VCF so each read overlapping informative heterozygous variants is annotated with an HP tag (HP:i:1 or HP:i:2); sort and index the tagged BAM for fast subset extraction. Split the tagged BAM into hp1.bam and hp2.bam, keeping only reads carrying HP:i:1 or HP:i:2 respectively. Rationale: haplotype assignment lets allele-specific CpG methylation states called per-molecule (e.g., by DeepBAM) be attributed to parental alleles. Compute the assignment rate as HP-tagged reads divided by total aligned reads to validate the split.

## Related tools

- **whatshap** (Runs haplotag to annotate each aligned read with HP:i:1 or HP:i:2 based on informative heterozygous variants it overlaps, using the GIAB phased VCF)
- **Oxford Nanopore Technology (ONT)** (Source platform of the HG002 R10.4 nanopore reads being haplotype-tagged)

## Examples

```
whatshap haplotag -o haplotagged.bam -r GRCh38.fa GIAB_phased.vcf.gz HG002_R10.4.bam
```

## Evaluation signals

- Confirm the haplotype-assignment rate is approximately 0.80 (the ~80% assignment rate reported for HG002 R10.4 data).
- Verify hp1.bam and hp2.bam exist and contain only consistently HP-tagged reads (HP:i:1 in hp1.bam, HP:i:2 in hp2.bam).
- Sum of reads in hp1.bam + hp2.bam equals the number of HP-tagged reads in the sorted, indexed haplotagged BAM.
- Untagged reads (no informative heterozygous variant overlap) should account for the remaining ~20% of aligned reads and are excluded from hp1/hp2 subsets.

## Limitations

- Reads that overlap no informative heterozygous GIAB variants receive no HP tag and cannot be assigned; the reported assignment rate is ~80%, so ~20% of reads are unassigned.
- Assignment quality depends on the correctness of the GIAB VCF phasing; misphased or missing heterozygous variants will misassign or drop reads.
- Only HP:i:1 and HP:i:2 reads are kept downstream, so allele-specific methylation analysis excludes unassignable reads.

## Evidence

- [other] Haplotype assignment was performed by tagging reads with whatshap using phased VCF files against mapped BAMs, then extracting reads corresponding to hp1 (HP:i:1) and hp2 (HP:i:2).: [evidence span withheld — anchor status: not_found]
