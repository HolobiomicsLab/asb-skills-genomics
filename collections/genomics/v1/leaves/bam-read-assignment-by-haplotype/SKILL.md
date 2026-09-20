---
name: bam-read-assignment-by-haplotype
description: Use when you have a mapped nanopore BAM plus a phased VCF for the same
  sample and need to separate reads into haplotype-specific sets, e.g. for allele-specific
  methylation analysis on HG002 R10.4 data.
when_to_use_negative:
- No phased VCF is available for the sample — haplotag depends on informative heterozygous
  variants to assign HP tags
- Input reads are not aligned to the same reference build as the phased VCF
- The goal is bulk CpG methylation frequency calling rather than allele-specific (haplotype-resolved)
  methylation analysis
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: whatshap
  role: Tags each aligned read with HP:i:1 or HP:i:2 based on overlapping informative
    heterozygous variants from the phased VCF
- name: Oxford Nanopore Technology (ONT)
  role: Source platform of the single-molecule reads being haplotype-assigned
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:cfadb319e8eaac5764409f8774696827e674862ef2e3ad99203482d4495bdb36
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# BAM read assignment by haplotype

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates mapped nanopore reads with paternal/maternal haplotype tags using whatshap haplotag against a phased VCF, then splits the tagged BAM into HP:i:1 and HP:i:2 read subsets for allele-specific methylation analysis.

## When to use

Use when you have a mapped nanopore BAM plus a phased VCF for the same sample and need to separate reads into haplotype-specific sets, e.g. for allele-specific methylation analysis on HG002 R10.4 data.

## When NOT to use

- No phased VCF is available for the sample — haplotag depends on informative heterozygous variants to assign HP tags
- Input reads are not aligned to the same reference build as the phased VCF
- The goal is bulk CpG methylation frequency calling rather than allele-specific (haplotype-resolved) methylation analysis

## Inputs

- Mapped Oxford Nanopore alignment BAM (e.g. HG002 R10.4)
- Genome In a Bottle whole-genome phased variant VCF (bgzipped/indexed)

## Outputs

- Haplotype-tagged, sorted and indexed BAM with HP:i:1/HP:i:2 read tags
- hp1.bam containing only HP:i:1 reads
- hp2.bam containing only HP:i:2 reads
- Haplotype-assignment rate (HP-tagged reads / total aligned reads)

## How to apply

Load the ONT alignment BAM and a GIAB whole-genome phased variant VCF for the same sample and reference build. Run `whatshap haplotag` with the phased VCF so each read overlapping informative heterozygous variants receives an integer tag HP:i:1 or HP:i:2. Sort and index the haplotype-tagged BAM so subsets can be extracted quickly, then split it into hp1.bam and hp2.bam keeping only reads carrying HP:i:1 or HP:i:2 respectively. Compute the haplotype-assignment rate as HP-tagged reads divided by total aligned reads and check it against the expected ~0.80 for HG002 R10.4 data before using the subsets in downstream methylation calling.

## Related tools

- **whatshap** (Tags each aligned read with HP:i:1 or HP:i:2 based on overlapping informative heterozygous variants from the phased VCF)
- **Oxford Nanopore Technology (ONT)** (Source platform of the single-molecule reads being haplotype-assigned)

## Examples

```
whatshap haplotag -o haplotagged.bam --output-tags HP GIAB_phased.vcf.gz HG002_ont_r104.bam; samtools sort -o haplotagged.sorted.bam haplotagged.bam; samtools index haplotagged.sorted.bam
```

## Evaluation signals

- Haplotype-assignment rate (HP-tagged reads / total aligned reads) is approximately 0.80, matching the reported ~80% assignment rate for HG002 R10.4 data
- hp1.bam and hp2.bam exist and contain only reads consistently tagged HP:i:1 and HP:i:2 respectively
- The haplotagged BAM is sorted and indexed so subsets can be extracted rapidly
- Reads are tagged only where they overlap informative heterozygous variants; untagged reads should correspond to regions without such variants

## Limitations

- Reads that do not overlap informative heterozygous variants remain untagged (~20% of aligned reads for HG002 R10.4)
- Requires a high-quality phased VCF (e.g. GIAB) matched to the sample and reference; assignment quality inherits phasing errors
- Assignment rate is sample- and dataset-dependent, so the ~0.80 benchmark is specific to HG002 R10.4 nanopore data

## Evidence

- [other] Haplotype assignment was performed by tagging reads with whatshap using phased VCF files against mapped BAMs, then extracting reads corresponding to hp1 (HP:i:1) and hp2 (HP:i:2).: [evidence span withheld — anchor status: not_found]
- [other] Run whatshap haplotag with the phased VCF to annotate each read with a haplotype tag (HP:i:1 or HP:i:2) based on informative heterozygous variants it overlaps.: [evidence span withheld — anchor status: not_found]
- [other] confirm the haplotype-assignment rate is approximately 0.80 (the ~80% assignment rate reported for HG002 R10.4 data), and verify hp1.bam and hp2.bam exist: [evidence span withheld — anchor status: not_found]
- [abstract] It enables unraveling allele-specific methylation patterns, including regions of transposable elements: "It enables unraveling allele-specific methylation patterns, including regions of transposable elements"
