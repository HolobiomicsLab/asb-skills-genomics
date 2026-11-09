---
name: binomial-model-snv-genotyping
description: Use when calling SNVs from ONT long-read pileups and candidate loci have
  already been classified as high-quality (known 1000G variants with R_alt > 0.3,
  or unknown variants with R_alt > 0.7 and no neighbor within 10 bp), and a statistical
  genotype assignment is needed.
when_to_use_negative:
- Candidate loci classified as low-quality (unknown variants with R_alt <= 0.7, low-complexity
  regions) — these require the haplotype-aware consensus/realignment pipeline instead
  of the binomial model.
- The task is indel or structural variant calling — miniSNV's binomial genotyping
  targets single nucleotide variants only.
- A pre-trained deep-learning caller is preferred for maximum accuracy on well-characterized
  human data — the statistical binomial approach trades DL model capacity for lightness
  and cross-species generalization.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3569
tools:
- name: miniSNV
  role: Implements the binomial likelihood genotyping of high-quality loci (theta
    = 0.03, equal priors over 0/0, 0/1, 1/1) within its SNV-calling pipeline
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: WhatsHap
  role: Consumes heterozygotes (quality > 15) emitted by the binomial genotyping step
    to phase and haplotag alignments; also supplies hapGT used in the preGT/hapGT
    consistency filter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:b1cee452fa5a59b7b2203478ef1279f9f2fb7ada8c38e35a22332ea369d2da08
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# binomial-model-snv-genotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

miniSNV assigns genotypes to high-quality candidate SNV loci by comparing the likelihoods of the three diploid genotypes (0/0, 0/1, 1/1) under a binomial model with sequencing error probability theta = 0.03, using allele counts from the read pileup rather than a trained deep-learning model.

## When to use

Use when calling SNVs from ONT long-read pileups and candidate loci have already been classified as high-quality (known 1000G variants with R_alt > 0.3, or unknown variants with R_alt > 0.7 and no neighbor within 10 bp), and a statistical genotype assignment is needed.

## When NOT to use

- Candidate loci classified as low-quality (unknown variants with R_alt <= 0.7, low-complexity regions) — these require the haplotype-aware consensus/realignment pipeline instead of the binomial model.
- The task is indel or structural variant calling — miniSNV's binomial genotyping targets single nucleotide variants only.
- A pre-trained deep-learning caller is preferred for maximum accuracy on well-characterized human data — the statistical binomial approach trades DL model capacity for lightness and cross-species generalization.

## Inputs

- Sorted, indexed ONT BAM alignment file (low-MAPQ, supplementary, secondary, and unmapped reads removed)
- Pileup-derived candidate loci with alternative allele ratios (R_alt threshold 0.2)
- Prebuilt known common-variant index from 1000 Genomes population VCF (allele frequency > 5%)
- Reference genome FASTA (samtools indexed)

## Outputs

- Genotyped candidate set for high-quality loci written to candidates.bed
- preGT genotypes (0/0, 0/1, 1/1) with quality scores used downstream for heterozygote selection and hapGT consistency filtering

## How to apply

After miniSNV's pileup screening (filter MAPQ < 20, supplementary, secondary, and unmapped alignments; mark loci where alternative allele ratio R_alt exceeds 0.2) and classification against the prebuilt known common-variant index (1000 Genomes, allele frequency > 5%, UIDs of form wid_pos_ref_alt), genotype each high-quality locus with a binomial likelihood model. The model compares the likelihoods of genotypes 0/0, 0/1, and 1/1 using equal priors and a fixed sequencing error probability theta = 0.03, taking the maximum-likelihood genotype as the preGT. Genotyped candidates are written to a BED file (candidates.bed) and the heterozygotes with quality score > 15 then feed the WhatsHap phasing step. The rationale is that a lightweight statistical model avoids labeled training data and generalizes across species, unlike DL-based callers such as Clair3 or DeepVariant.

## Related tools

- **miniSNV** (Implements the binomial likelihood genotyping of high-quality loci (theta = 0.03, equal priors over 0/0, 0/1, 1/1) within its SNV-calling pipeline) — https://github.com/CuiMiao-HIT/miniSNV
- **WhatsHap** (Consumes heterozygotes (quality > 15) emitted by the binomial genotyping step to phase and haplotag alignments; also supplies hapGT used in the preGT/hapGT consistency filter)

## Examples

```
python chr_chunk_task.py --fin_ref hg38.fa --fin_bam sample.sorted.bam --workDir work --useindex -i index_folder --human
```

## Evaluation signals

- candidates.bed is produced containing genotypes only for high-quality loci (known variants with R_alt > 0.3; unknown variants with R_alt > 0.7 and no other candidate within 10 bp)
- Genotype likelihoods are evaluated only over the three diploid states 0/0, 0/1, 1/1 with error probability theta = 0.03 — outputs outside this state space indicate misuse
- Heterozygotes selected for phasing all carry quality scores > 15; lower-quality heterozygotes must be absent from the WhatsHap input
- Final bgzipped VCF (pipeline_output.vcf.gz) on HG002 should reach F1-score and recall > 99.7% against the GIAB truth set, per the paper's benchmark
- Variants with preGT 0/1 but hapGT 1/1 must be dropped by the downstream consistency filter — their presence indicates the filter was not applied

## Limitations

- The binomial model is only applied to high-quality loci; low-quality and low-complexity loci need the MSA/local-assembly consensus plus KSW2 realignment route
- Thresholds (R_alt 0.3/0.7 split) were selected via in-house experiments (Supplementary Table S1) and may not transfer to all error profiles
- Because it is a statistical, non-DL method, per-locus accuracy can differ from deep-learning callers, though benchmarks showed comparable or higher F1-scores on ONT data

## Evidence

- [methods] miniSNV assigns the genotypes for high-quality loci by comparing the likelihoods of possible genotypes using a binomial model: "miniSNV assigns the genotypes for high-quality loci by comparing the likelihoods of possible genotypes using a binomial model"
- [methods] If the locus is found and >0.3, it is classified as high-quality. If the locus is not found but meets the following two conditions, it also qualifies as high-quality: [evidence span withheld — anchor status: not_found]
- [methods] The combination of threshold of 0.3–0.7 was selected according to a series of in-house experiments shown in Supplementary Table S1: [evidence span withheld — anchor status: not_found]
- [methods] miniSNV initially selects heterozygote variants with quality scores exceeding 15 from a high-quality variant set.: "miniSNV initially selects heterozygote variants with quality scores exceeding 15 from a high-quality variant set."
- [readme] miniSNV is available from https://github.com/CuiMiao-HIT/miniSNV: "miniSNV takes the known common variants in the population as variation backgrounds and leverages read pileup, read-based phasing, and consensus generation to discover and genotype SNVs for ONT long"
