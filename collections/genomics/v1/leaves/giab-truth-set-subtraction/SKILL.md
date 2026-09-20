---
name: giab-truth-set-subtraction
description: Use when a variant call set must be partitioned against a reference truth
  set — e.g. retaining Clairvoyante calls supported by both PacBio and ONT HG001 data
  but absent from GIAB v3.3.2.
when_to_use_negative:
- Accuracy benchmarking where truth-set variants must be kept to classify true positives/false
  negatives (use a genotype-aware RTG vcfeval comparison against GIAB v3.3.2 instead)
- When only a single sequencing technology is available — the catalog design requires
  concordant PacBio AND ONT support, so one-platform calls cannot pass the concordance
  filter
- When multi-allelic SNPs or indels >4 bp are of interest — Clairvoyante excludes
  multi-allelic SNPs and provides exact alternative alleles only for indels <=4 bp,
  so such candidates cannot enter the catalog
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Clairvoyante
  role: Tensorflow-based multi-task convolutional neural network that generates the
    PacBio and ONT call sets (variant type, zygosity, alternative allele, indel length)
    that are filtered and intersected before truth-set subtraction
  repo: https://github.com/aquaskyline/Clairvoyante
- name: RTG vcfeval
  role: Genotype-aware comparison engine used to classify calls against the GIAB v3.3.2
    HG001 truth set in the paper's benchmarks; the same truth VCF is the subtracted
    baseline in this skill
- name: Genome-in-a-Bottle (GIAB) v3.3.2 HG001
  role: High-quality truth dataset whose variant set is subtracted from the concordant
    PacBio+ONT call set
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:855613aaa49ccd98cb2df78dbc2f9479edb0a7728614dba8a4eba7cb31620803
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Genome-in-a-Bottle (GIAB) v3.3.2 HG001
  - Illumina
  - RTG vcfeval
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# giab-truth-set-subtraction

## Summary

Subtracts GIAB v3.3.2 HG001 truth-set variants from concordant long-read (PacBio + ONT) Clairvoyante call sets to isolate candidate variants absent from the truth set, yielding a 3,135-variant catalog (2,732 SNPs, 298 deletions, 105 insertions; Ti/Tv ~1.42) of sites missed by Illumina.

## When to use

Use when a variant call set must be partitioned against a reference truth set — e.g. retaining Clairvoyante calls supported by both PacBio and ONT HG001 data but absent from GIAB v3.3.2.

## When NOT to use

- Accuracy benchmarking where truth-set variants must be kept to classify true positives/false negatives (use a genotype-aware RTG vcfeval comparison against GIAB v3.3.2 instead)
- When only a single sequencing technology is available — the catalog design requires concordant PacBio AND ONT support, so one-platform calls cannot pass the concordance filter
- When multi-allelic SNPs or indels >4 bp are of interest — Clairvoyante excludes multi-allelic SNPs and provides exact alternative alleles only for indels <=4 bp, so such candidates cannot enter the catalog

## Inputs

- Genome-wide HG001 Clairvoyante VCF call sets from PacBio reads (0.2 AF cutoff)
- Genome-wide HG001 Clairvoyante VCF call sets from ONT reads (rel3, R9.4 chemistry, aligned to GRCh37 decoy v5 with NGMLR 0.2.3)
- GIAB version 3.3.2 HG001 truth VCF
- GRCh37 reference used for alignment and coordinate comparison

## Outputs

- supplementary_data1_variants.tsv — variant list absent from the GIAB truth set (3,135 variants)
- variant_summary.txt — composition counts: 2,732 SNPs, 298 deletions, 105 insertions, Ti/Tv ~1.42

## How to apply

Load the genome-wide HG001 Clairvoyante call sets generated from PacBio and Oxford Nanopore reads at the 0.2 allele-frequency cutoff and filter each to variants with geometric-mean variant quality >= 135. Intersect the two filtered sets to keep only positions concordantly supported by both single-molecule technologies — this dual-platform support is the safeguard against long-read artifacts. Subtract the GIAB v3.3.2 HG001 truth variants (the paper's truth dataset, evaluated genotype-aware in benchmarks via RTG vcfeval) so only concordant calls absent from the truth set remain. Classify the retained variants into SNPs, insertions, and deletions and compute the transition/transversion (Ti/Tv) ratio as a sanity check on catalog composition. Write the variant list as supplementary_data1_variants.tsv and the summary counts as variant_summary.txt.

## Related tools

- **Clairvoyante** (Tensorflow-based multi-task convolutional neural network that generates the PacBio and ONT call sets (variant type, zygosity, alternative allele, indel length) that are filtered and intersected before truth-set subtraction) — https://github.com/aquaskyline/Clairvoyante
- **RTG vcfeval** (Genotype-aware comparison engine used to classify calls against the GIAB v3.3.2 HG001 truth set in the paper's benchmarks; the same truth VCF is the subtracted baseline in this skill)
- **Genome-in-a-Bottle (GIAB) v3.3.2 HG001** (High-quality truth dataset whose variant set is subtracted from the concordant PacBio+ONT call set)

## Evaluation signals

- Catalog size invariant: retained variants must total 3,135 = 2,732 SNPs + 298 deletions + 105 insertions; any deviation indicates the intersection or subtraction step changed
- Ti/Tv ratio of the retained SNP catalog should be ~1.42, consistent with the published candidate catalog
- No retained variant should overlap GIAB v3.3.2 HG001 truth positions — the subtraction must be complete
- Every retained variant must be present in both the PacBio and ONT call sets after filtering at geometric-mean variant quality >= 135 and 0.2 AF cutoff
- Output files must follow Supplementary Data 1 format (supplementary_data1_variants.tsv) plus variant_summary.txt

## Limitations

- Only candidates absent from GIAB v3.3.2 can be recovered — variants present in the truth set are discarded regardless of long-read support
- Clairvoyante ignores variants with two or more alternative alleles and excludes multi-allelic SNPs from training, so such sites cannot enter the catalog
- Exact alternative alleles are provided only for indels <=4 bp due to scarce long-indel training samples, skewing the indel composition of the catalog
- ONT data are available only for HG001, limiting cross-sample generalization of the concordance-based catalog

## Evidence

- [abstract] 3,135 variants missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads: "we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads"
- [discussion] GIAB v3.3.2 HG001 used as truth dataset: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] Reads re-aligned to GRCh37 decoy with NGMLR for the long-read call sets: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] Multi-allelic SNPs excluded by Clairvoyante, bounding the catalog: "In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training"
