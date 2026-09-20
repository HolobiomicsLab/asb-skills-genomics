---
name: kmer-scored-consensus-error-correction
description: Use when polishing a high-quality (e.g., QV > 50) HiFi/hifiasm assembly
  that still carries consensus errors, especially in repeats or gap-filled ONT regions,
  and overcorrection or switch errors must be avoided.
when_to_use_negative:
- The assembly is a draft built from noisy long reads without supplementary short-read
  k-mer data — performance relies heavily on short reads to check whether a K-mer
  contains errors
- The goal is de novo assembly or large structural-variant correction; NextPolish2
  targets residual SNV/indel consensus errors in already-high-quality assemblies
- Trio-binned assemblies where reads of the other haplotype were not discarded before
  mapping — different-haplotype reads should be removed prior to alignment
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0196
tools:
- name: NextPolish2
  role: Core polishing engine that corrects SNV/indel errors and runs the repeat-aware
    phasing module (LQR + Louvain grouping via -r)
  repo: https://github.com/Nextomics/NextPolish2
- name: Winnowmap
  role: Maps HiFi reads to the draft assembly using weighted minimizers with a repetitive
    k-mer list (-W repetitive_k15.txt)
  repo: https://github.com/marbl/Winnowmap
- name: meryl
  role: Counts k=15 k-mers of the assembly and prints k-mers with distinct>0.9998
    as the repetitive k-mer list for winnowmap
  repo: https://github.com/marbl/meryl
- name: yak
  role: Builds k21 and k31 k-mer databases (-b37) from >=60X short reads used by NextPolish2
    to judge erroneous k-mers
  repo: https://github.com/lh3/yak
- name: minimap2
  role: Alternative aligner (map-pb) for preparing the HiFi mapping file if winnowmap
    is not used
  repo: https://github.com/lh3/minimap2/
- name: fastp
  role: Quality control and filtering of short reads (adapter removal, trimming, -q
    20) before yak k-mer counting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:f43c04e4eedf7dd1a5e24dc647f17684863733266050f6eee0e5f06b098ab76d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - NextPolish2
  - Winnowmap
  - fastp
  - meryl
  - minimap2
  - yak
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# kmer-scored-consensus-error-correction

## Summary

A k-mer-guided genome polishing workflow (NextPolish2) that corrects residual SNV/indel errors in HiFi-assembled T2T genomes using HiFi read alignments plus yak k-mer datasets, while the built-in phasing module (LQR detection with Louvain community grouping) maintains haplotype consistency and avoids overcorrection in repeats.

## When to use

Use when polishing a high-quality (e.g., QV > 50) HiFi/hifiasm assembly that still carries consensus errors, especially in repeats or gap-filled ONT regions, and overcorrection or switch errors must be avoided.

## When NOT to use

- The assembly is a draft built from noisy long reads without supplementary short-read k-mer data — performance relies heavily on short reads to check whether a K-mer contains errors
- The goal is de novo assembly or large structural-variant correction; NextPolish2 targets residual SNV/indel consensus errors in already-high-quality assemblies
- Trio-binned assemblies where reads of the other haplotype were not discarded before mapping — different-haplotype reads should be removed prior to alignment

## Inputs

- Draft genome assembly FASTA (e.g., asm.fa.gz from hifiasm on ~30x PacBio HiFi reads)
- PacBio HiFi long reads (FASTQ/FASTA, e.g., hifi.fasta.gz)
- High-coverage (>=60X recommended) Illumina short reads for yak k-mer counting
- Repetitive k-mer list (repetitive_k15.txt) from meryl for winnowmap -W

## Outputs

- Polished assembly FASTA (asm.np2.fa)
- Sorted/indexed HiFi read alignment BAM (hifi.map.sort.bam)
- yak k-mer databases (k21.yak, k31.yak)
- Validation metrics: Merqury QV, DeepVariant homozygous high-quality variant counts, Paftools.js switch-error statistics

## How to apply

Align HiFi reads to the draft assembly with winnowmap (building a repetitive k-mer list first via meryl count k=15 and printing k-mers with distinct>0.9998 to repetitive_k15.txt passed through -W), then sort/index with samtools to get hifi.map.sort.bam. Quality-filter short reads with fastp (adapter removal, trimming -5 -3 -f 5 -F 5 -t 5 -T 5, -q 20) and build two yak k-mer databases from >=60X short reads: yak count -o k21.yak -k 21 -b 37 and yak count -o k31.yak -k 31 -b 37. Run nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa; short-read k-mers are used to decide whether a K-mer contains errors, so correction confidence depends on their quality. Re-run with the -r flag to enable the repeat-aware phasing module, which groups reads spanning the same LQR by haplotype or repeat copy via Louvain community detection and, for conflicting communities, by default keeps the community sharing the most k-mers with the reference, preserving haplotype consistency. Validate with Merqury QV, DeepVariant homozygous high-quality variant counts, and Paftools.js switch-error checks, benchmarked against the HG005 result (QV 53.8544 -> 55.6257; homozygous high-quality variants 14,155 -> 6,955).

## Related tools

- **NextPolish2** (Core polishing engine that corrects SNV/indel errors and runs the repeat-aware phasing module (LQR + Louvain grouping via -r)) — https://github.com/Nextomics/NextPolish2
- **Winnowmap** (Maps HiFi reads to the draft assembly using weighted minimizers with a repetitive k-mer list (-W repetitive_k15.txt)) — https://github.com/marbl/Winnowmap
- **meryl** (Counts k=15 k-mers of the assembly and prints k-mers with distinct>0.9998 as the repetitive k-mer list for winnowmap) — https://github.com/marbl/meryl
- **yak** (Builds k21 and k31 k-mer databases (-b37) from >=60X short reads used by NextPolish2 to judge erroneous k-mers) — https://github.com/lh3/yak
- **minimap2** (Alternative aligner (map-pb) for preparing the HiFi mapping file if winnowmap is not used) — https://github.com/lh3/minimap2/
- **fastp** (Quality control and filtering of short reads (adapter removal, trimming, -q 20) before yak k-mer counting)

## Examples

```
winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz | samtools sort -o hifi.map.sort.bam - ; ./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- Merqury QV of the polished assembly exceeds the draft (HG005 benchmark: 53.8544 -> 55.6257)
- DeepVariant homozygous high-quality variant counts (potential errors) decrease without collapsing to near zero, indicating correction without overcorrection (HG005: 14,155 -> 6,955)
- Switch-error rate in heterozygous regions does not increase and ideally slightly decreases after enabling the built-in phasing module
- Corrected assembly retains reference haplotype in conflicting repeat communities (community sharing the most k-mers with the reference is kept by default)
- Sorted BAM (hifi.map.sort.bam) and both yak databases (k21.yak, k31.yak) complete without errors, confirming prerequisite k-mer datasets were built from >=60X short reads

## Limitations

- Performance relies heavily on short reads to check whether a K-mer contains errors; uncorrected errors caused by short-read sequencing biases are a risk without high-coverage short reads (>=60X recommended)
- Without the -r repeat-aware phasing module, haplotype/repeat-copy grouping is not performed and switch-error behavior is not optimized
- No software version numbers are stated for NextPolish2 or its dependencies in the paper, complicating reproducibility
- Default conflict resolution keeps the community sharing the most k-mers with the reference, so highly divergent alternative haplotypes may be biased toward the reference sequence

## Evidence

- [methods] NextPolish2 improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955.: "NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955."
- [intro] Through the built-in phasing module, it can not only correct the error bases, but also maintain the original haplotype consistency.: "Through the built-in phasing module, it can not only correct the error bases, but also maintain the original haplotype consistency. In fact, our evaluation shows that it even slightly reduces switch"
- [discussion] The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors.: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [methods] ./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [methods] meryl count k=15 output merylDB asm.fa.gz and meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt: "meryl count k=15 output merylDB asm.fa.gz
meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt"
- [intro] NextPolish2 can fix base errors in “highly accurate” draft assemblies without introducing overcorrections, even in regions with highly repetitive elements: "NextPolish2 can fix base errors in “highly accurate” draft assemblies without introducing overcorrections, even in regions with highly repetitive elements"
