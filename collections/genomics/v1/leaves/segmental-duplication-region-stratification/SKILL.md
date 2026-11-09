---
name: segmental-duplication-region-stratification
description: Use when comparing long-read vs short-read SNV call sets and you need
  to show whether calls in highly similar segmental duplications differ in number
  and transition/transversion quality between platforms.
when_to_use_negative:
- The goal is accurate short indel calling from SMS reads — the article states accurate
  calling of short indels is challenging due to the high insertion/deletion error
  rate, so stratified SNV analysis will not address it.
- Input reads are already uniquely mappable short reads and the question does not
  concern repetitive/duplicated regions — stratification by duplication identity adds
  no signal.
- The variant call sets have not been filtered by the study's GQ, density, and depth
  rules — intersecting unfiltered VCFs would confound platform comparison with caller-specific
  false-positive clusters.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Calls SNVs from the 30x PacBio SMS BAM (default min_cov 6, max_cov 8000, min_mapq
    20, min_alt_count 3, min_alt_frac 0.125) to produce the long-read call set stratified
    against duplication tracks
  repo: https://github.com/pjedge/longshot
- name: FreeBayes
  role: Produces the short-read Illumina SNV call set on the 33x BWA-MEM-aligned BAM
    used as the platform comparison baseline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:5fa4fd2873900dc3c1dbab560212a8e9590057aea9813683fc718f3c20900094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - FreeBayes
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# segmental-duplication-region-stratification

## Summary

Stratify SNV call sets by segmental-duplication identity thresholds (≥95% and ≥99%) and compute per-stratum variant counts and Ts/Tv ratios to quantify how poorly short reads map in duplicated regions versus SMS long reads.

## When to use

Use when comparing long-read vs short-read SNV call sets and you need to show whether calls in highly similar segmental duplications differ in number and transition/transversion quality between platforms.

## When NOT to use

- The goal is accurate short indel calling from SMS reads — the article states accurate calling of short indels is challenging due to the high insertion/deletion error rate, so stratified SNV analysis will not address it.
- Input reads are already uniquely mappable short reads and the question does not concern repetitive/duplicated regions — stratification by duplication identity adds no signal.
- The variant call sets have not been filtered by the study's GQ, density, and depth rules — intersecting unfiltered VCFs would confound platform comparison with caller-specific false-positive clusters.

## Inputs

- Longshot VCF of SNVs called from 30x PacBio SMS BAM aligned to hs37d5
- FreeBayes VCF of SNVs called from 33x Illumina BAM aligned to hs37d5 with BWA-MEM
- GIAB high-confidence region BED tracks and their complements
- Segmental-duplication annotation tracks with ≥95% and ≥99% identity thresholds

## Outputs

- Stratified SNV counts per caller per region stratum (e.g. 180,889 vs 116,649 at ≥95%; 78,851 vs 18,684 at ≥99%)
- Ts/Tv ratios per stratum and caller, including the Illumina-only Ts/Tv in segmental duplications (reported 1.55 vs 1.99)

## How to apply

Intersect both call sets (e.g. Longshot on 30x PacBio reads aligned to hs37d5 with BLASR, and FreeBayes on 33x Illumina reads aligned with BWA-MEM) with segmental-duplication annotation tracks restricted to ≥95% and ≥99% identity thresholds, alongside GIAB high-confidence regions and their complements. Apply the study's genotype-quality filters: fixed GQ cutoff of 50 for short-read (FreeBayes) calls and a variable GQ cutoff matched to the median read coverage for Longshot, plus the density filter (>10 SNVs in a 500 bp window) and the read-depth filter d+5*sqrt(d). Compute per-stratum SNV counts and Ts/Tv ratios separately for each caller, including the Illumina-only Ts/Tv within segmental duplications. The rationale is that Illumina Ts/Tv drops (~1.55 at ≥99% identity) in duplicated regions due to poor mappability, while SMS-based calls retain a genome-wide-like ratio (~1.99), demonstrating long-read advantage where short reads cannot uniquely align.

## Related tools

- **Longshot** (Calls SNVs from the 30x PacBio SMS BAM (default min_cov 6, max_cov 8000, min_mapq 20, min_alt_count 3, min_alt_frac 0.125) to produce the long-read call set stratified against duplication tracks) — https://github.com/pjedge/longshot
- **FreeBayes** (Produces the short-read Illumina SNV call set on the 33x BWA-MEM-aligned BAM used as the platform comparison baseline)

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- Reproduce the Table 2 pattern: 180,889 SMS SNVs vs 116,649 Illumina SNVs at ≥95% identity (55.0% more with SMS), and 78,851 vs 18,684 (4.2-fold) at ≥99% identity.
- Illumina-only Ts/Tv in ≥99% identity segmental duplications should be ≈1.55, versus ≈1.99 for SMS-only calls.
- Verify the density filter removed clusters of >10 SNVs in a 500 bp window and the depth filter removed variants with read depth > d+5*sqrt(d).
- Confirm GQ filtering was asymmetric as specified: fixed GQ 50 for FreeBayes calls, variable GQ matched to median read coverage for Longshot calls.
- Check that both call sets were intersected with GIAB high-confidence regions and their complements so strata are defined on the same accessible genome space.

## Limitations

- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, so long-read calls in these regions are not guaranteed error-free.
- GIAB and Platinum Genomes variant sets were generated using short-read datasets and are therefore biased in favor of short-read technologies, which can inflate apparent Longshot false positives.
- Some Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets.
- Allele observations with phred-scaled quality below the default 7.0 threshold are discarded, reducing the effective read depth for SMS reads.

## Evidence

- [abstract] Longshot enables variant detection in duplicated regions of the genome that cannot be mapped using short reads: "enables variant detection in duplicated regions of the genome that cannot be mapped using short reads"
- [discussion] Variable GQ cutoff for Longshot: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] Density and depth filters: [evidence span withheld — anchor status: not_found]
- [discussion] PacBio mapping challenge in segmental duplications: "However, correctly mapping PacBio reads in highly similar segmental duplications remains a challenge."
- [readme] Longshot CLI from README: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
- [discussion] GIAB high-confidence benchmark data: [evidence span withheld — anchor status: not_found]
