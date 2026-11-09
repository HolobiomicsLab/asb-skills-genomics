---
name: snv-calling-from-ont-long-reads
description: Use when you have minimap2-aligned ONT (or PacBio SMRT) whole-genome
  BAM/CRAM reads plus an indexed GRCh38/hg38 FASTA and need accurate, phased diploid
  SNV calls benchmarked against GIAB high-confidence sets.
when_to_use_negative:
- You need to call indels de novo — Longshot only calls SNVs; it can genotype indels
  only if supplied in an input VCF, and accurate short indel calling from SMS reads
  is limited by high insertion/deletion error rates.
- Structural variants > 50 bp are needed — they are not supported by Longshot and
  require SV-specific callers.
- Input is short-read sequencing data — the haplotype-informed genotyping model is
  designed for long, error-prone SMS reads, not Illumina data.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: 'Primary SNV caller: performs candidate SNV identification by genotype likelihood
    from the pileup, pair-HMM local realignment to ref/alt, and haplotype-informed
    phased genotyping via HapCUT2.'
  repo: github.com/pjedge/longshot
- name: HapCUT2
  role: Internal phasing engine used within Longshot's iterative optimization of haplotypes
    and local genotype updates.
- name: Genome in a Bottle (GIAB)
  role: High-confidence truth call set used to compute precision, recall, and F1 of
    the ONT SNV calls.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:5b49f37c7450e9f673875d443c5cd77dc5f3c5e028f3985fff14e2ec043aa834
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genome in a Bottle (GIAB)
  - HapCUT2
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# snv-calling-from-ont-long-reads

## Summary

Call and phase diploid SNVs from Oxford Nanopore (or PacBio) long, error-prone reads using Longshot, which combines pileup-based allelotyping, pair-HMM local realignment, and haplotype-informed genotyping via HapCUT2. On NA12878 37x whole-genome ONT data this reproduces best F1 0.967 against GIAB high-confidence calls in ~5h13m on one core.

## When to use

Use when you have minimap2-aligned ONT (or PacBio SMRT) whole-genome BAM/CRAM reads plus an indexed GRCh38/hg38 FASTA and need accurate, phased diploid SNV calls benchmarked against GIAB high-confidence sets.

## When NOT to use

- You need to call indels de novo — Longshot only calls SNVs; it can genotype indels only if supplied in an input VCF, and accurate short indel calling from SMS reads is limited by high insertion/deletion error rates.
- Structural variants > 50 bp are needed — they are not supported by Longshot and require SV-specific callers.
- Input is short-read sequencing data — the haplotype-informed genotyping model is designed for long, error-prone SMS reads, not Illumina data.

## Inputs

- Sorted, indexed BAM/CRAM of ONT reads aligned to the reference (e.g. minimap2-aligned NA12878 37x ONT whole-genome BAM)
- Indexed FASTA reference genome (e.g. GRCh38/hg38)
- Optional tabix-indexed gzipped VCF of potential variants to genotype instead of pileup discovery
- GIAB high-confidence call set for the matching reference build (for accuracy validation)

## Outputs

- Phased VCF of called SNVs with genotype qualities (longshot --out)
- Optional haplotype-tagged BAM with HP:i:1/HP:i:2 read assignments (--out_bam)
- Accuracy metrics table (precision, recall, F1 vs GIAB) e.g. na12878_ont_accuracy.tsv

## How to apply

Install Longshot via Bioconda ('conda install longshot'; dependencies include Rust >= 1.40.0, zlib, xz, clang). Run on the sorted, indexed ONT BAM against the GRCh38 reference FASTA ('longshot --bam ont.bam --ref grch38.fa --out na12878_ont_longshot.vcf'), optionally processing chromosomes separately with --region (<chrom>:<start>-<stop>). Keep the default filters tuned for error-prone SMS reads: min_mapq 20, min_allele_qual 7.0, min_cov 6, and the strand-bias Fisher's exact test cutoff (--strand_bias_pvalue_cutoff 0.01) that removes SNVs over-represented on one strand. Keep the default depth filter, which removes variants with read depth > d + 5*sqrt(d) (median depth d) to exclude copy-number-amplified regions; use --auto_max_cov to compute coverage automatically. Optionally export phased reads with -O to get HP:i:1/HP:i:2 haplotype tags, then benchmark calls against the GIAB high-confidence set with a variable GQ cutoff matched to the median read coverage (GQ 65 for the NA12878 37x ONT run), computing precision, recall, and F1 from contingency counts.

## Related tools

- **Longshot** (Primary SNV caller: performs candidate SNV identification by genotype likelihood from the pileup, pair-HMM local realignment to ref/alt, and haplotype-informed phased genotyping via HapCUT2.) — github.com/pjedge/longshot
- **HapCUT2** (Internal phasing engine used within Longshot's iterative optimization of haplotypes and local genotype updates.)
- **Genome in a Bottle (GIAB)** (High-confidence truth call set used to compute precision, recall, and F1 of the ONT SNV calls.)

## Examples

```
longshot --bam ont.bam --ref grch38.fa --out na12878_ont_longshot.vcf --strand_bias_pvalue_cutoff 0.01 --min_mapq 20 --min_allele_qual 7.0
```

## Evaluation signals

- Against GIAB high-confidence calls on GRCh38 with GQ 65 (median-coverage-matched), expect precision ~0.991, recall ~0.933, best F1 0.967 on NA12878 ONT data.
- Runtime check: whole-genome ONT variant calling should complete in ~5h13m using a single core.
- Installation sanity check: the example_data toy run (30x simulated PacBio BAM, 3x200kb contigs, 714 true variants) completes in ~5–10 seconds and can be compared to ground_truth_variants.vcf.
- Output VCF must be phased and contain genotype-quality fields; with -O, reads receive HP:i:1/HP:i:2 tags.
- Fisher's exact strand-bias filter (p < 0.01, two-tailed) and max-cov cutoff d + 5*sqrt(d) should exclude strand-biased and copy-number-amplified sites.

## Limitations

- Longshot only calls SNVs; indels can be genotyped but not discovered, and accurate short indel calling with SMS reads is challenging due to the high insertion/deletion error rate.
- Correctly mapping long reads in highly similar segmental duplications remains a challenge.
- GIAB and Platinum Genomes truth sets were generated with short reads and are biased in favor of short-read technologies, so some Longshot false positives may correspond to false negatives in the GIAB call sets.
- The default allele quality cutoff (7.0) discards low-quality allele observations, reducing effective read depth for SMS reads.
- Excessive false potential variants supplied via -v/--potential_variants can lead to inaccurate genotyping results.

## Evidence

- [results] Longshot achieved a best F1 score of 0.967 and took only 5 h and 13 min for variant calling using a single core.: [evidence span withheld — anchor status: not_found]
- [discussion] Strand bias filter p<0.01: [evidence span withheld — anchor status: not_found]
- [discussion] Depth filter d+5√d: [evidence span withheld — anchor status: not_found]
- [discussion] Variable GQ cutoff: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [abstract] Longshot algorithm description: "We introduce a variant calling method, Longshot, which leverages the haplotype information present in SMS reads to accurately detect and phase single-nucleotide variants (SNVs) in diploid genomes."
- [readme] Longshot usage: "$ longshot [FLAGS] [OPTIONS] --bam <BAM/CRAM> --ref <FASTA> --out <VCF>"
- [readme] Phased read output tags: "Write new bam file with haplotype tags (HP:i:1 and HP:i:2) for reads assigned to each haplotype, any existing HP and PS tags are removed"
- [readme] Longshot SNV-only scope: "Currently, it only calls single nucleotide variants (SNVs), but it can genotype indels if they are given in an input VCF."
