---
name: variant-call-quality-filtering
description: Use when you have genome-wide Clairvoyante call sets from two single-molecule
  sequencing technologies (e.g. PacBio and ONT) and need to retain only concordant,
  high-quality variants absent from a GIAB truth set.
when_to_use_negative:
- Your goal is de novo variant calling from raw BAMs — use Clairvoyante/Clair calling
  (callVarBam.py) first; this skill only filters and reconciles existing call sets.
- Your variants of interest are already present in the GIAB truth set — the subtraction
  step explicitly discards truth-set variants.
- You need multi-allelic variants or long indels (>4 bp exact alleles) — Clairvoyante
  ignores two-or-more alternative alleles and reports exact alleles only for indels
  <=4 bp.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: Clairvoyante
  role: Source of the variant call sets being filtered; its per-variant quality scores
    drive the geometric-mean quality >= 135 cutoff
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Clair
  role: Successor to Clairvoyante for generating the germline small variant calls
    that this filtering workflow reconciles
  repo: https://github.com/HKU-BAL/Clair
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:e871ee7b84e335c43ea17327e7cf450b261a7c04b522d2f7f4dfcac046dd2de0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
  - Clairvoyante
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# variant-call-quality-filtering

## Summary

Filter Clairvoyante variant calls from PacBio and Oxford Nanopore HG001 data by a geometric-mean variant quality threshold (>= 135) at a 0.2 allele-frequency cutoff, intersect the two technology call sets, subtract the GIAB v3.3.2 truth set, and characterize the residual catalog of candidate Illumina-missed variants.

## When to use

Use when you have genome-wide Clairvoyante call sets from two single-molecule sequencing technologies (e.g. PacBio and ONT) and need to retain only concordant, high-quality variants absent from a GIAB truth set.

## When NOT to use

- Your goal is de novo variant calling from raw BAMs — use Clairvoyante/Clair calling (callVarBam.py) first; this skill only filters and reconciles existing call sets.
- Your variants of interest are already present in the GIAB truth set — the subtraction step explicitly discards truth-set variants.
- You need multi-allelic variants or long indels (>4 bp exact alleles) — Clairvoyante ignores two-or-more alternative alleles and reports exact alleles only for indels <=4 bp.

## Inputs

- Genome-wide Clairvoyante VCF-style call set from PacBio HG001 reads at 0.2 allele-frequency cutoff
- Genome-wide Clairvoyante call set from Oxford Nanopore (MinION rel3, R9.4 chemistry) HG001 reads at 0.2 cutoff
- GIAB v3.3.2 HG001 truth variant set

## Outputs

- supplementary_data1_variants.tsv — TSV of 3,135 candidate variants absent from GIAB
- variant_summary.txt — summary of 2,732 SNPs, 298 deletions, 105 insertions, Ti/Tv ~ 1.42

## How to apply

Load the genome-wide HG001 Clairvoyante call sets generated from PacBio and Oxford Nanopore reads at the 0.2 cutoff. Filter each call set to variants with a geometric-mean variant quality >= 135, then intersect the two filtered sets to keep only positions concordantly supported by both technologies. Subtract GIAB v3.3.2 HG001 truth variants so only calls absent from the truth set remain, yielding a catalog of 3,135 candidate Illumina-missed variants. Classify the retained calls into SNPs (2,732), deletions (298), and insertions (105), and compute the transition/transversion (Ti/Tv) ratio (~1.42) as a sanity check on catalog quality. Write the variant list as supplementary_data1_variants.tsv and the summary counts as variant_summary.txt.

## Related tools

- **Clairvoyante** (Source of the variant call sets being filtered; its per-variant quality scores drive the geometric-mean quality >= 135 cutoff) — https://github.com/aquaskyline/Clairvoyante
- **Clair** (Successor to Clairvoyante for generating the germline small variant calls that this filtering workflow reconciles) — https://github.com/HKU-BAL/Clair

## Evaluation signals

- After subtracting GIAB v3.3.2, exactly 3,135 concordant PacBio+ONT variants remain genome-wide
- Catalog composition matches 2,732 SNPs, 298 deletions, and 105 insertions
- Computed Ti/Tv ratio of the retained catalog is approximately 1.42
- Every retained position is supported in both the PacBio and ONT filtered call sets (intersection non-empty per position)
- No variant in the output carries a geometric-mean variant quality below 135 or falls below the 0.2 allele-frequency cutoff

## Limitations

- Clairvoyante excludes multi-allelic SNPs and ignores variants with two or more alternative alleles, so such variants cannot survive this filtering workflow.
- Exact alternative alleles are only provided for indels <=4 bp due to scarcity of long-indel training samples, limiting indel characterization in the catalog.
- ONT data are available only for HG001, restricting cross-sample validation of the concordant candidate catalog.

## Evidence

- [abstract] we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads: "we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads"
- [discussion] GIAB v3.3.2 truth set used for subtraction: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [intro] Clairvoyante substantially outperforms Nanopolish and DeepVariant on PacBio and ONT data: "Clairvoyante is on par with GATK UnifiedGenotyper on Illumina data, and substantially outperforms Nanopolish and DeepVariant on PacBio and ONT data"
- [discussion] Indel F1 for ONT remains ~50%, limiting indel confidence: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
- [discussion] Reads realigned to GRCh37 with NGMLR before calling: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
