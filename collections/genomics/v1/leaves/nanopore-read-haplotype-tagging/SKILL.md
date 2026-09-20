---
name: nanopore-read-haplotype-tagging
description: Use when ONT nanopore reads (e.g., an HG002 R10.4 alignment BAM) must
  be partitioned into paternal/maternal haplotype read sets for allele-specific methylation
  analysis and a phased VCF of heterozygous variants is available.
when_to_use_negative:
- No phased VCF of heterozygous variants exists for the sample — haplotag has no informative
  variants to assign HP:i:1/HP:i:2 tags from.
- The BAM is already haplotype-tagged; only the split/filter step (extracting hp1.bam
  and hp2.bam) is needed, not re-tagging.
- The goal is single-molecule CpG methylation calling itself (e.g., running DeepBAM
  or Dorado on R10.4 POD5/BAM data) rather than assigning reads to haplotypes.
edam_operation: http://edamontology.org/operation_0488
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3407
tools:
- name: whatshap
  role: Annotates each aligned ONT read with a haplotype tag (HP:i:1 or HP:i:2) from
    the phased VCF, enabling split into hp1/hp2 read sets
- name: DeepBAM
  role: Downstream R10.4 CpG methylation caller applied per haplotype read set for
    allele-specific methylation analysis
  repo: https://github.com/huicongyao/DeepBAM.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:b5cb7159e420481e7a901f0d69e3d56bb7bc8cc0636a8949ebf4273bbb1abfab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM.git
  tools:
  - DeepBAM
  - Oxford Nanopore Technology (ONT)
  - whatshap
  edam_operation: http://edamontology.org/operation_0488
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# nanopore-read-haplotype-tagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns Oxford Nanopore reads to paternal or maternal haplotypes by tagging them with whatshap haplotag against GIAB phased variants (HP:i:1 / HP:i:2), as done for HG002 R10.4 data in DeepBAM to enable allele-specific methylation analysis at ~80% assignment rate.

## When to use

Use when ONT nanopore reads (e.g., an HG002 R10.4 alignment BAM) must be partitioned into paternal/maternal haplotype read sets for allele-specific methylation analysis and a phased VCF of heterozygous variants is available.

## When NOT to use

- No phased VCF of heterozygous variants exists for the sample — haplotag has no informative variants to assign HP:i:1/HP:i:2 tags from.
- The BAM is already haplotype-tagged; only the split/filter step (extracting hp1.bam and hp2.bam) is needed, not re-tagging.
- The goal is single-molecule CpG methylation calling itself (e.g., running DeepBAM or Dorado on R10.4 POD5/BAM data) rather than assigning reads to haplotypes.

## Inputs

- Oxford Nanopore R10.4 alignment BAM (e.g., HG002 reads)
- GIAB whole-genome phased variant VCF (informative heterozygous variants)

## Outputs

- Haplotype-tagged BAM with HP:i:1 / HP:i:2 read tags, sorted and indexed
- hp1.bam and hp2.bam: per-haplotype read subsets
- Haplotype-assignment rate metric (HP-tagged reads / total aligned reads)

## How to apply

Load the HG002 Oxford Nanopore R10.4 alignment BAM together with the GIAB whole-genome phased variant VCF and run `whatshap haplotag`, which annotates each read with an HP tag (HP:i:1 or HP:i:2) based on the informative heterozygous variants it overlaps. Sort and index the haplotype-tagged BAM so per-haplotype subsets can be extracted quickly. Split the tagged BAM into hp1.bam and hp2.bam, keeping only reads carrying HP:i:1 or HP:i:2 respectively; these read sets feed downstream per-haplotype CpG methylation calling (e.g., with DeepBAM) to reveal allele-specific methylation, including in transposable-element regions. Judge success by the haplotype-assignment rate — HP-tagged reads divided by total aligned reads — which should be approximately 0.80 for HG002 R10.4 data, and by confirming hp1.bam and hp2.bam contain only consistently HP-tagged reads.

## Related tools

- **whatshap** (Annotates each aligned ONT read with a haplotype tag (HP:i:1 or HP:i:2) from the phased VCF, enabling split into hp1/hp2 read sets)
- **DeepBAM** (Downstream R10.4 CpG methylation caller applied per haplotype read set for allele-specific methylation analysis) — https://github.com/huicongyao/DeepBAM.git

## Examples

```
whatshap haplotag -o haplotagged.bam GIAB_phased.vcf.gz HG002_R10.4.bam
```

## Evaluation signals

- Haplotype-assignment rate (HP-tagged reads / total aligned reads) is approximately 0.80, matching the reported rate for HG002 R10.4 data.
- hp1.bam and hp2.bam exist and contain only reads with consistent HP:i:1 or HP:i:2 tags respectively.
- No read appears in both hp1.bam and hp2.bam (HP tags are mutually exclusive per read).
- The haplotype-tagged BAM is coordinate-sorted and indexed so subset extraction is fast.
- Each HP tag traces to overlapping informative heterozygous variants in the GIAB phased VCF.

## Limitations

- Reads that overlap no informative heterozygous variant remain untagged; only ~80% of HG002 R10.4 reads are assigned, leaving ~20% unassigned.
- Assignment quality depends on the correctness of the GIAB phased VCF — phasing errors propagate to read haplotype labels.
- Validated on human HG002 R10.4 data with GIAB phasing; no parameter guidance is given for other genomes or species.

## Evidence

- [other] Haplotype assignment via whatshap on GIAB phased VCF: [evidence span withheld — anchor status: not_found]
- [other] Split into per-haplotype read sets: [evidence span withheld — anchor status: not_found]
