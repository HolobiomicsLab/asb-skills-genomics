---
name: long-read-variant-phasing-haplotagging
description: Use when calling SNVs from ONT long-read BAM alignments where low-quality
  candidate loci must be resolved via haplotype-aware consensus sequences, or when
  phased, haplotagged alignments are needed to build per-haplotype local assemblies.
when_to_use_negative:
- Input is short-read Illumina data — the method relies on long ONT reads spanning
  multiple variants for phasing and haplotagging
- All candidate loci are already classified high-quality and confidently genotyped
  by the binomial model, so haplotype-aware consensus generation adds no information
- The goal is structural variant or indel calling — the pipeline targets SNVs only
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: Pipeline driver that performs pileup screening, locus classification, and
    haplotype-aware genotyping around the phasing step
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: WhatsHap
  role: Phases heterozygous SNVs (quality > 15) and haplotags reads to generate the
    haplotype-specific phased alignment (v2.0)
- name: abPOA
  role: MSA-based consensus model used on haplotagged read subsets when >70% of reads
    are spanning reads
- name: KSW2
  role: Realignment of generated haplotype consensus sequences against the local reference
    to derive alleles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:3e99d7788b0301e72ff1ef66b4072e39d3a16f5fd7f97f8f0822dc962a75b90e
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
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# long-read-variant-phasing-haplotagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Phase heterozygous SNVs from ONT long reads with WhatsHap and haplotag alignments so that haplotype-specific read subsets can be used for consensus generation and haplotype-aware genotyping, as done in the miniSNV pipeline.

## When to use

Use when calling SNVs from ONT long-read BAM alignments where low-quality candidate loci must be resolved via haplotype-aware consensus sequences, or when phased, haplotagged alignments are needed to build per-haplotype local assemblies.

## When NOT to use

- Input is short-read Illumina data — the method relies on long ONT reads spanning multiple variants for phasing and haplotagging
- All candidate loci are already classified high-quality and confidently genotyped by the binomial model, so haplotype-aware consensus generation adds no information
- The goal is structural variant or indel calling — the pipeline targets SNVs only

## Inputs

- Sorted, samtools-indexed ONT BAM alignment filtered to remove MAPQ < 20, supplementary, secondary, and unmapped reads
- Samtools-indexed FASTA reference genome
- High-quality heterozygote variant set with quality score > 15 (e.g. candidates.bed / pre-genotyped VCF)
- Prebuilt known common-variant index (1000 Genomes, allele frequency > 5%)

## Outputs

- Haplotype-specific phased alignment (phased.bam) with haplotag PS/HP tags
- Valid tag-1/tag-2 phasing sets with read counts > 2
- Per-haplotype local consensus sequences (consensus.fa) for active regions (locus +/- 100 bp)
- hapGT haplotype-aware genotypes for low-quality loci, merged into the final bgzipped VCF (pipeline_output.vcf.gz)

## How to apply

First screen the MAPQ>=20 pileup (R_alt > 0.2) and genotype high-quality loci with the binomial model (theta = 0.03) to obtain a candidate set. Select heterozygote variants with quality scores exceeding 15 from the high-quality variant set and run WhatsHap (v2.0) to phase them and haplotag the raw reads, producing a haplotype-specific phased alignment (phased.bam). Reads are grouped into tag-0/1/2 phasing sets; tag-1 and tag-2 sets are valid only if their read counts exceed the default threshold of 2. For each active region (candidate locus +/- 100 bp), build up to two local haplotype consensuses using the abPOA MSA model when >70% of reads are spanning reads (overlap ratio > 0.6), otherwise de Bruijn assembly with k from 31 to 121 in 10 steps (overlap ratio >= 0.4). The rationale is that haplotagging separates alleles that pileup screening alone cannot disentangle in low-quality or low-complexity regions.

## Related tools

- **miniSNV** (Pipeline driver that performs pileup screening, locus classification, and haplotype-aware genotyping around the phasing step) — https://github.com/CuiMiao-HIT/miniSNV
- **WhatsHap** (Phases heterozygous SNVs (quality > 15) and haplotags reads to generate the haplotype-specific phased alignment (v2.0))
- **abPOA** (MSA-based consensus model used on haplotagged read subsets when >70% of reads are spanning reads)
- **KSW2** (Realignment of generated haplotype consensus sequences against the local reference to derive alleles)

## Examples

```
python chr_chunk_task.py --fin_ref hg38.fa --fin_bam sample.bam --workDir work/ --useindex -i index_folder --human
```

## Evaluation signals

- phased.bam exists and contains reads carrying HP/PS haplotag values with tag-1 and tag-2 sets each exceeding 2 supporting reads
- consensus.fa contains at most two haplotypes per active region (candidate locus +/- 100 bp)
- Haplotypes with fewer than three supporting reads are absent from the downstream realignment/genotyping step
- preGT/hapGT consistency filter applied: variants where preGT is 0/1 but hapGT is 1/1 are dropped as systematic-error false positives
- Final VCF (pipeline_output.vcf.gz) is bgzipped and, on HG002 sup-datasets, achieves F1-score and recall > 99.7% against the GIAB truth set

## Limitations

- abPOA was chosen because FMAlign and HAlign, while performant, 'seem not support to produce multiple consensus sequences at heterozygous sites'
- A preGT of 0/1 combined with hapGT of 1/1 indicates alternative alleles randomly scattered across haplotypes by systematic errors and must be filtered
- R_alt thresholds 0.3–0.7 for high-quality locus classification were tuned by in-house experiments and may need re-validation on other datasets
- Assembly-based fallback (k-mers 31–121) is only triggered when the overlap ratio is >= 0.4; highly fragmented coverage below this may fail consensus generation

## Evidence
