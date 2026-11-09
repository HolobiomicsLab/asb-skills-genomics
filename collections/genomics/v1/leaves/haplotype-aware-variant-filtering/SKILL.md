---
name: haplotype-aware-variant-filtering
description: Use when calling SNVs from ONT long reads at loci classified as low-quality
  or low-complexity, where per-locus binomial genotyping is unreliable and haplotype-level
  consensus evidence must be generated, realigned, genotyped, and filtered.
when_to_use_negative:
- Loci already classified as high-quality and genotyped by the binomial likelihood
  model (theta = 0.03, priors over 0/0, 0/1, 1/1) — these skip the haplotype consensus
  route
- Short-read (Illumina-style) data — the method's phasing and consensus steps depend
  on ONT long-read spanning alignments
- Repetitive regions without sufficient overlapping reads — with <40% overlap ratio
  and few spanning reads, neither the MSA nor assembly model has adequate support
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3673
tools:
- name: miniSNV
  role: 'Implements the haplotype-aware filtering pipeline: consensus generation,
    KSW2 realignment, hapGT assignment, and preGT/hapGT consistency filtering'
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: WhatsHap
  role: Phases heterozygotes (quality > 15) and haplotags raw reads to produce the
    phased alignment whose read counts validate phasing sets
- name: abPOA
  role: Multiple sequence alignment engine used to produce multiple haplotype consensus
    sequences when >70% of reads are spanning reads
- name: KSW2
  role: Aligns surviving haplotype consensus sequences to the local reference part
    for allele discovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:3ec262dc6a7cc39a825cae6fc56ace8140799b2c963b9a4e591903e143227b05
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - KSW2
  - WhatsHap
  - abPOA
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# haplotype-aware-variant-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

miniSNV's post-consensus variant filtering stage for low-quality loci: haplotype consensus sequences (abPOA MSA or de Bruijn assembly) are realigned with KSW2, hapGT genotypes are assigned, and variants are dropped when phasing support is weak or when preGT is 0/1 but hapGT is 1/1, to avoid systematic-error false positives.

## When to use

Use when calling SNVs from ONT long reads at loci classified as low-quality or low-complexity, where per-locus binomial genotyping is unreliable and haplotype-level consensus evidence must be generated, realigned, genotyped, and filtered.

## When NOT to use

- Loci already classified as high-quality and genotyped by the binomial likelihood model (theta = 0.03, priors over 0/0, 0/1, 1/1) — these skip the haplotype consensus route
- Short-read (Illumina-style) data — the method's phasing and consensus steps depend on ONT long-read spanning alignments
- Repetitive regions without sufficient overlapping reads — with <40% overlap ratio and few spanning reads, neither the MSA nor assembly model has adequate support

## Inputs

- Haplotype-specific phased alignment (phased.bam) from WhatsHap phasing/haplotagging
- Candidate loci BED (candidates.bed) from high-quality binomial genotyping
- Candidate haplotype consensus sequences (consensus.fa) from MSA or local assembly
- Samtools-indexed reference FASTA (local reference window per active region)

## Outputs

- Haplotype-filtered, realigned variant calls with hapGT genotypes
- Merged final SNV callset as bgzipped VCF (pipeline_output.vcf.gz)

## How to apply

After WhatsHap phasing and haplotagging produce phased alignments, extract reads in active regions (candidate locus +/- 100 bp) and generate up to two haplotype consensus sequences: use the abPOA-based MSA model when >70% of reads are spanning reads (overlap ratio > 0.6), otherwise de Bruijn assembly with k-mers from 31 to 121. Filter haplotypes with fewer than three supporting reads, then align the remaining consensus sequences to the local reference with KSW2 and annotate discovered alleles within 10 bp. Assign hapGT genotypes and apply the consistency filter: drop any variant where preGT is '0/1' but hapGT is '1/1', since scattered alternative alleles across both haplotypes indicate systematic errors rather than true heterozygosity. Merge the surviving high- and low-quality genotype results into a single bgzipped VCF.

## Related tools

- **miniSNV** (Implements the haplotype-aware filtering pipeline: consensus generation, KSW2 realignment, hapGT assignment, and preGT/hapGT consistency filtering) — https://github.com/CuiMiao-HIT/miniSNV
- **WhatsHap** (Phases heterozygotes (quality > 15) and haplotags raw reads to produce the phased alignment whose read counts validate phasing sets)
- **abPOA** (Multiple sequence alignment engine used to produce multiple haplotype consensus sequences when >70% of reads are spanning reads)
- **KSW2** (Aligns surviving haplotype consensus sequences to the local reference part for allele discovery)

## Examples

```
python chr_chunk_task.py --fin_ref FIN_REF --fin_bam FIN_BAM --workDir WORKDIR
```

## Evaluation signals

- Every retained low-quality variant has a haplotype supported by at least three reads after the support filter
- No variant in the final callset has preGT = 0/1 with hapGT = 1/1 (all such variants must be dropped)
- Phasing sets 1 and 2 each have read counts exceeding the threshold (default > 2) in phased.bam
- Each active region (candidate locus +/- 100 bp) yields at most two consensus haplotypes in consensus.fa
- Final merged VCF (pipeline_output.vcf.gz) is bgzipped and contains SNV records matching GIAB truth at F1/recall > 99.7% on HG002 sup-data

## Limitations

- Haplotypes with fewer than three supporting reads are discarded, so rare variants at very low coverage can be missed
- The preGT 0/1 vs hapGT 1/1 filter removes variants where alternative-allele reads scatter randomly across both haplotypes due to systematic errors, but may also drop true hom-alt calls preceded by inconsistent preGT
- abPOA was chosen because FMAlign and HAlign do not support producing multiple consensus sequences at heterozygous sites, constraining MSA tool substitution

## Evidence
