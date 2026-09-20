---
name: correct-then-assemble-pipeline-execution
description: Use when assembling a diploid genome from long noisy reads (Nanopore
  R9/R10 or PacBio CLR) and you need haplotype-aware error correction that retains
  heterozygous SNP alleles, or when benchmarking heterozygote retention at low heterozygosity
  rates.
when_to_use_negative:
- 'Input reads are already highly accurate (Nanopore duplex or PacBio HiFi): the paper
  reports PECAT achieves only mediocre performance on these data types'
- 'The goal is SNP calling from corrected reads alone without a raw-read caller: even
  after error correction, SNP accuracy is still low in low heterozygosity rate regions
  (corrected-read accuracy 92-94% vs 96-98% for raw reads at heterozygosity 0.0001)'
- 'Only limited coverage is available and a high-quality phased assembly is required:
  PECAT needs higher coverage of data'
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: PECAT
  role: Runs the haplotype-aware error correction (`pecat.pl correct`) and two-round
    diploid string graph assembly; provides the SNP calling criteria applied to corrected
    reads
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Maps corrected reads to the first-round assembly (e.g. '-x map-pb -c -p 0.5
    -r 1000' for PacBio CLR, '-x ava-pb' for candidate overlaps between PacBio CLR
    raw reads)
  repo: https://github.com/lh3/minimap2
- name: Pomoxis
  role: Reference-based evaluation and simulation of diploid reference genomes and
    noisy long reads for heterozygote-retention benchmarking
  repo: https://github.com/nanoporetech/pomoxis
- name: Clair3
  role: Optional re-calling of SNPs on raw Nanopore reads to supplement corrected-read
    SNP calls
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Optional consensus polishing to further improve assembly quality
  repo: https://github.com/nanoporetech/medaka
- name: racon
  role: Consensus/polishing dependency required by the PECAT pipeline (v1.4.21+)
  repo: https://github.com/lbcb-sci/racon
- name: samtools
  role: SAM/BAM processing dependency of the PECAT pipeline (1.7+)
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:5bbdd620b99ca21e8bfb1ee8d6565a82f9ed3b13f0c5316aa777376c75a85dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - Medaka
  - PECAT
  - Pomoxis
  - minimap2
  - racon
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# correct-then-assemble-pipeline-execution

## Summary

Execute PECAT's correct-then-assemble pipeline, combining haplotype-aware error correction with a two-round string graph assembly to reconstruct diploid genomes from long noisy reads (Nanopore R9/R10, PacBio CLR). The skill tests whether correction retains heterozygote alleles, especially in low-heterozygosity regions where SNP accuracy degrades.

## When to use

Use when assembling a diploid genome from long noisy reads (Nanopore R9/R10 or PacBio CLR) and you need haplotype-aware error correction that retains heterozygous SNP alleles, or when benchmarking heterozygote retention at low heterozygosity rates.

## When NOT to use

- Input reads are already highly accurate (Nanopore duplex or PacBio HiFi): the paper reports PECAT achieves only mediocre performance on these data types
- The goal is SNP calling from corrected reads alone without a raw-read caller: even after error correction, SNP accuracy is still low in low heterozygosity rate regions (corrected-read accuracy 92-94% vs 96-98% for raw reads at heterozygosity 0.0001)
- Only limited coverage is available and a high-quality phased assembly is required: PECAT needs higher coverage of data

## Inputs

- Long noisy raw reads (FASTQ/FASTA) from Nanopore R9, Nanopore R10, or PacBio CLR at roughly >=80X coverage (PECAT extracts the longest 80X raw reads, or all reads if the dataset is <80X)
- Diploid reference genome with simulated heterozygous SNPs (for the Pomoxis-based benchmarking variant of this workflow)

## Outputs

- Error-corrected reads with retained heterozygote alleles
- Haplotype-collapsed genome from the first assembly round
- Heterozygous SNP calls on corrected reads (coverage range [10, 1000]; second-most-common-base thresholds r_l/r_h/c_l/c_h = 0.4/0.2/10/100)
- Phased haplotype-specific contigs (two-round string graph assembly)
- Per-allele SNP accuracy table (e.g. snp_accuracy_0.0001.csv) for benchmarked heterozygosity rates

## How to apply

Install PECAT via `git clone --recursive https://github.com/lemene/PECAT.git` (or the docker image hub.docker.com/r/lemene/pecat); it requires python3, minimap2 (2.17+), racon (1.4.21+), perl, and samtools, with Clair3 (v0.1-r12+) and Medaka as optional dependencies. Run the haplotype-aware error correction (POA graph based) with `pecat.pl correct`, which retains heterozygote alleles while correcting sequencing errors; for Nanopore data, call SNPs again on raw reads with Clair3 because corrected reads alone are insufficient for accurate SNP calling. PECAT then generates a haplotype-collapsed genome, maps corrected reads back with minimap2 (e.g. 'map-pb -c -p 0.5 -r 1000' for PacBio CLR), and calls heterozygous SNPs using the coverage-range [10, 1000] filter and the second-most-common-base criteria with defaults r_l/r_h/c_l/c_h = 0.4/0.2/10/100. To probe heterozygote retention, simulate a diploid reference at a target heterozygosity rate (e.g. 0.0001) with Pomoxis reference-based simulation, correct the simulated noisy reads, re-call SNPs, and compare against truth variants. Reuse first-round overlaps, remove inconsistent ones (strict corrected-read threshold: 8 different SNP alleles; loose raw-read threshold: 6), and reassemble; polish with Medaka for Nanopore data.

## Related tools

- **PECAT** (Runs the haplotype-aware error correction (`pecat.pl correct`) and two-round diploid string graph assembly; provides the SNP calling criteria applied to corrected reads) — https://github.com/lemene/PECAT
- **minimap2** (Maps corrected reads to the first-round assembly (e.g. '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR, '-x ava-pb' for candidate overlaps between PacBio CLR raw reads)) — https://github.com/lh3/minimap2
- **Pomoxis** (Reference-based evaluation and simulation of diploid reference genomes and noisy long reads for heterozygote-retention benchmarking) — https://github.com/nanoporetech/pomoxis
- **Clair3** (Optional re-calling of SNPs on raw Nanopore reads to supplement corrected-read SNP calls) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Optional consensus polishing to further improve assembly quality) — https://github.com/nanoporetech/medaka
- **racon** (Consensus/polishing dependency required by the PECAT pipeline (v1.4.21+)) — https://github.com/lbcb-sci/racon
- **samtools** (SAM/BAM processing dependency of the PECAT pipeline (1.7+)) — https://github.com/samtools/samtools

## Examples

```
git clone --recursive https://github.com/lemene/PECAT.git && pecat.pl correct reads.fq
```

## Evaluation signals

- SNP allele accuracy at heterozygosity 0.0001 should exceed the baseline where corrected-read calling (92-94%) drops below raw-read level (96-98%)
- Called heterozygous SNP sites satisfy the coverage-range [10, 1000] filter and the second-most-common-base count criteria with defaults r_l/r_h/c_l/c_h = 0.4/0.2/10/100
- Overlaps marked inconsistent during reassembly meet the strict corrected-read threshold of 8 different SNP alleles, or the loose raw-read threshold of 6 different SNP alleles
- For Nanopore datasets, raw-read SNP calls from Clair3 are available as a cross-check against corrected-read calls
- Assembly quality benchmarks (e.g. with Pomoxis reference-based evaluation, hapdiff for SVs) reproduce haplotype-resolved contiguity comparable to the paper's HG002 Nanopore R10 phase block NG50 of 59.4/58.0 Mb

## Limitations

- The accuracy of SNP calls from corrected reads is still low for low heterozygosity rate regions; at heterozygosity 0.0001 corrected reads (92-94%) fall below raw reads (96-98%)
- The error correction method may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions, even with highly accurate reads; resolution is deferred to subsequent PECAT releases
- PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads
- PECAT needs higher coverage of data to obtain high-quality phased assemblies
- Detailed assembly parameters are only described in Supplementary Notes 4 and 5, not in the main text, limiting reproducibility

## Evidence

- [abstract] PECAT, a Phased Error Correction and Assembly Tool, for reconstructing diploid genomes from long noisy reads: "Here, we present PECAT, a Phased Error Correction and Assembly Tool, for reconstructing diploid genomes from long noisy reads."
- [abstract] design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors: "We design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors."
- [discussion] PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X: "For each of the above data, PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X, for error correction and assembly."
- [discussion] Its coverage is in the range [10, 1000].: "(1) Its coverage is in the range [10, 1000]."
- [discussion] the accuracy of this SNP call is still low for low heterozygosity rate regions: "Nevertheless, the accuracy of this SNP call is still low for low heterozygosity rate regions."
- [discussion] even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [discussion] we use Clair3 to call SNPs again for Nanopore reads: "Therefore, we use Clair3 to call SNPs again for Nanopore reads."
- [methods] git clone --recursive https://github.com/lemene/PECAT.git: "git clone --recursive https://github.com/lemene/PECAT.git"
- [methods] There is a pre-built docker image: "There is a pre-built [docker image](https://hub.docker.com/r/lemene/pecat)."
- [discussion] We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
