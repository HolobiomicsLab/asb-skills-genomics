---
name: msa-consensus-haplotype-generation
description: Use when calling SNVs from ONT long-read BAMs at low-quality candidate
  loci (low-complexity regions, unknown variants), where a pileup-based genotype alone
  is unreliable and per-haplotype consensus sequences are needed to call and genotype
  variants.
when_to_use_negative:
- The locus was already classified as high-quality (known variant with R_alt > 0.3
  or unknown variant with R_alt > 0.7 and no other candidate within 10 bp) — these
  are genotyped directly with the binomial model and bypass consensus generation
- Data is short-read or non-ONT, since the skill's spanning-read and overlap-ratio
  decision points (70% spanning, overlap > 0.6/0.4) are calibrated for long nanopore
  reads
- Only single-consensus output is needed — FMAlign/HAlign-style MSA that cannot produce
  multiple consensus sequences at heterozygous sites cannot replace this diploid-aware
  step
edam_operation: http://edamontology.org/operation_0300
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Orchestrates active-region extraction, consensus model selection (MSA vs local
    assembly), realignment, and haplotype-aware genotyping for low-quality loci
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: WhatsHap
  role: Phases high-quality heterozygotes and haplotags reads to supply haplotype-specific
    alignments (phased.bam) that feed consensus generation
- name: abPOA
  role: Performs the multiple sequence alignment used to build consensus sequences
    when >70% of reads are spanning reads
- name: KSW2
  role: Realigns filtered haplotype consensus sequences against the local reference
    to identify alternative alleles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:ae35093f1a26324bd9c39ba15e6343039bb1e808cc0dc66eb325e1e1fe6eee94
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
  edam_operation: http://edamontology.org/operation_0300
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# msa-consensus-haplotype-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate haplotype consensus sequences at candidate SNV loci by extracting overlapping reads and building either an abPOA-based multiple sequence alignment or a de Bruijn-graph local assembly, then realigning the resulting consensus sequences to the local reference to derive alternative alleles for low-quality loci in miniSNV.

## When to use

Use when calling SNVs from ONT long-read BAMs at low-quality candidate loci (low-complexity regions, unknown variants), where a pileup-based genotype alone is unreliable and per-haplotype consensus sequences are needed to call and genotype variants.

## When NOT to use

- The locus was already classified as high-quality (known variant with R_alt > 0.3 or unknown variant with R_alt > 0.7 and no other candidate within 10 bp) — these are genotyped directly with the binomial model and bypass consensus generation
- Data is short-read or non-ONT, since the skill's spanning-read and overlap-ratio decision points (70% spanning, overlap > 0.6/0.4) are calibrated for long nanopore reads
- Only single-consensus output is needed — FMAlign/HAlign-style MSA that cannot produce multiple consensus sequences at heterozygous sites cannot replace this diploid-aware step

## Inputs

- Haplotype-specific phased alignment (phased.bam) from WhatsHap phasing/haplotagging
- Candidate locus set (candidates.bed) from binomial-model genotyping of high-quality loci
- Samtools-indexed reference FASTA (local reference sequences for active regions)
- Samtools-indexed ONT long-read BAM filtered for MAPQ >= 20, non-supplementary, non-secondary, mapped reads

## Outputs

- Haplotype consensus sequences per active region (consensus.fa)
- Haplotype-aware genotypes (hapGT) with preGT/hapGT consistency filtering
- Merged SNV callset as bgzipped VCF (pipeline_output.vcf.gz)

## How to apply

After WhatsHap phasing/haplotagging of heterozygotes (quality > 15, phasing-set read counts > 2), define active regions around candidate loci (+/- 100 bp) and choose a consensus model per region: use the abPOA-based MSA model when more than 70% of reads are spanning reads (overlap ratio > 0.6), otherwise fall back to de Bruijn graph assembly with k increasing from 31 to 121 in 10 steps (overlap ratio >= 0.4). Generate up to two haplotype-specific consensus sequences per region and write them to consensus.fa; abPOA is preferred because FMAlign and HAlign do not support producing multiple consensus sequences at heterozygous sites. Filter haplotypes with fewer than three supporting reads, realign surviving consensus sequences to the local reference with KSW2, annotate discovered alleles within 10 bp, and assign hapGT genotypes. Apply the preGT/hapGT consistency filter (drop variants where preGT is 0/1 but hapGT is 1/1) to avoid false positives from systematic errors, then merge with high-quality pileup genotypes into the final bgzipped VCF.

## Related tools

- **miniSNV** (Orchestrates active-region extraction, consensus model selection (MSA vs local assembly), realignment, and haplotype-aware genotyping for low-quality loci) — https://github.com/CuiMiao-HIT/miniSNV
- **WhatsHap** (Phases high-quality heterozygotes and haplotags reads to supply haplotype-specific alignments (phased.bam) that feed consensus generation)
- **abPOA** (Performs the multiple sequence alignment used to build consensus sequences when >70% of reads are spanning reads)
- **KSW2** (Realigns filtered haplotype consensus sequences against the local reference to identify alternative alleles)

## Examples

```
python chr_chunk_task.py --fin_ref FIN_REF --fin_bam FIN_BAM --workDir WORKDIR
```

## Evaluation signals

- consensus.fa contains up to two haplotype consensus sequences per active region (candidate locus +/- 100 bp), matching the diploid model
- Every emitted haplotype is backed by at least three supporting reads; haplotypes below this threshold must have been dropped
- No variant in the final VCF has preGT 0/1 with hapGT 1/1 (the consistency filter removes these)
- Pipeline completes all stages without error and the output pipeline_output.vcf.gz contains SNV records on HG002; per the paper, F1-score and recall should exceed 99.7% against the GIAB truth set on sup-datasets

## Limitations

- Low-complexity-region candidates are always classified as low-quality, so consensus generation carries the genotyping burden there; errors in MSA or assembly propagate to hapGT
- The preGT=0/1 but hapGT=1/1 pattern indicates reads with alternative alleles randomly scattered across haplotypes (possible systematic error) and such variants are filtered, so genuine variants with this pattern are lost
- When fewer than 70% of reads are spanning reads, the de Bruijn graph fallback (k 31–121 in 10 steps, overlap ratio >= 0.4) is used, which is more sensitive to assembly failure than the MSA path

## Evidence
