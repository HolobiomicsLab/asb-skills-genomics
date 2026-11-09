---
name: cohort-scale-variant-count-aggregation
description: Reference for reproducing or benchmarking a cohort-wide DNV total from
  HAT/HAT-FLEX trio output, e.g., aggregating per-trio calls from thousands of SSC
  or similar WGS/WES trios and checking DNV quality metrics.
when_to_use_negative:
- You only have raw BAM/CRAM files and no access to run the Hare/Tortoise variant-calling
  workflow upstream — HAT-FLEX on existing VCFs would be the appropriate entry point
  instead of aggregating HAT trio output.
- Your data is not parent-child trio sequencing output (e.g., singletons or duo samples)
  — the custom de novo variant filter requires a trio to distinguish child DNVs from
  inherited variants.
- Your variants have not yet been filtered as de novo (i.e., you have raw joint-genotyped
  VCFs) — aggregation must occur only after HAT's custom DNV filter.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: HAT (Hare And Tortoise)
  role: Produces the per-trio DNV calls that are aggregated into the cohort-level
    count
  repo: https://github.com/TNTurnerLab/HAT
- name: Hare
  role: GPU-accelerated trio variant-calling workflow (Parabricks, GATK HaplotypeCaller,
    DeepVariant, GLnexus) whose filtered output feeds the cohort aggregation
  repo: https://github.com/TNTurnerLab/Hare
- name: GLnexus
  role: Forms family-level joint-genotyped files that are run through HAT's custom
    de novo variant filter before aggregation
- name: squirrel.py
  role: Converts HAT output into the tab-delimited format required as acorn input
    for cohort summarization
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package for reading the aggregated DNV file (readDNV) and summarizing cohort
    DNV characteristics including CpG percentage
  repo: https://github.com/TNTurnerLab/acorn
- name: HAT-FLEX
  role: Drop-in trio DNV detection tool that operates directly on existing VCFs, an
    alternative source of per-trio DNV calls for aggregation
  repo: https://github.com/TNTurnerLab/HAT-FLEX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:5112a008000a2172250eeaeeb11e242b195224e50998fea0bc5583b4b181f213
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - DeepVariant
  - GATK
  - GLnexus
  - HAT
  - HAT (Hare And Tortoise)
  - HAT-FLEX
  - Hare
  - Parabricks
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cohort-scale-variant-count-aggregation

## Summary

Aggregates per-trio de novo variant (DNV) calls from the HAT workflow into a cohort-level count table (e.g., 329,589 DNVs across 4216 Simons Simplex Collection short-read WGS trios) and validates callset quality using metrics such as DNVs per individual, percent at CpG sites, and percent phased to the paternal chromosome of origin.

## When to use

Reference for reproducing or benchmarking a cohort-wide DNV total from HAT/HAT-FLEX trio output, e.g., aggregating per-trio calls from thousands of SSC or similar WGS/WES trios and checking DNV quality metrics.

## When NOT to use

- You only have raw BAM/CRAM files and no access to run the Hare/Tortoise variant-calling workflow upstream — HAT-FLEX on existing VCFs would be the appropriate entry point instead of aggregating HAT trio output.
- Your data is not parent-child trio sequencing output (e.g., singletons or duo samples) — the custom de novo variant filter requires a trio to distinguish child DNVs from inherited variants.
- Your variants have not yet been filtered as de novo (i.e., you have raw joint-genotyped VCFs) — aggregation must occur only after HAT's custom DNV filter.

## Inputs

- Per-trio HAT DNV call outputs (VCF or HAT DNV text files) from the 4216 Simons Simplex Collection parent-child trios
- Family-level GLnexus joint-genotyped VCFs passed through HAT's custom de novo variant filter
- HAT output prepped to acorn input format via squirrel.py (tab-delimited: sample, chromosome with 'chr' prefix, genomic position, reference allele, alternate allele)
- CpG annotation rda file (e.g., cpg_b38.rda) for CpG-site summarization with acorn

## Outputs

- Cohort-level DNV count table (e.g., total 329,589 de novo variants across 4216 trios)
- Per-individual DNV counts and QC metric summaries (DNVs per individual, percent at CpG sites, percent phased to paternal chromosome of origin)
- acorn DNV object for follow-up statistics (SNV/INDEL/MNV extraction, Ti/Tv ratio, deletion/insertion ratio, CpG annotation)

## How to apply

Run the Hare (Parabricks v4.0.0-1 with GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0, GLnexus joint genotyping) or Tortoise workflow on each trio, then pass family-level joint-genotyped files through HAT's custom de novo variant filter to extract DNVs present in the child but absent from both parents. Aggregate the per-trio DNV call outputs into a cohort-level count table, summing DNV counts across all trios (the SSC 4216-trio reference total is 329,589 de novo variants). For downstream statistics, convert HAT output to acorn input format with squirrel.py from the HAT repo, then load it into the R package acorn via readDNV (tab-delimited sample, chr-prefixed chromosome, position, reference allele, alternate allele) and summarize with functions like annotateCpG. Correct application is judged by whether the cohort total matches the published count and whether DNVs per individual, percent of DNVs at CpG sites, and percent phased to the paternal chromosome of origin are consistent with published expectations.

## Related tools

- **HAT (Hare And Tortoise)** (Produces the per-trio DNV calls that are aggregated into the cohort-level count) — https://github.com/TNTurnerLab/HAT
- **Hare** (GPU-accelerated trio variant-calling workflow (Parabricks, GATK HaplotypeCaller, DeepVariant, GLnexus) whose filtered output feeds the cohort aggregation) — https://github.com/TNTurnerLab/Hare
- **GLnexus** (Forms family-level joint-genotyped files that are run through HAT's custom de novo variant filter before aggregation)
- **squirrel.py** (Converts HAT output into the tab-delimited format required as acorn input for cohort summarization) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package for reading the aggregated DNV file (readDNV) and summarizing cohort DNV characteristics including CpG percentage) — https://github.com/TNTurnerLab/acorn
- **HAT-FLEX** (Drop-in trio DNV detection tool that operates directly on existing VCFs, an alternative source of per-trio DNV calls for aggregation) — https://github.com/TNTurnerLab/HAT-FLEX

## Examples

```
python squirrel.py --input hat_dnvs.txt; R CMD INSTALL acorn_0.99.9.tar.gz; Rscript -e "library(acorn); d <- readDNV('hat_dnvs_acorn.txt'); CpGresult <- annotateCpG(DNVobject = d)"
```

## Evaluation signals

- Cohort-wide DNV total matches the published 329,589 de novo variants for the 4216 SSC short-read WGS trios
- DNVs per individual falls within the range of published values (roughly 60-80 DNVs per child expected for WGS trios)
- Percent of DNVs at CpG sites is consistent with published expectations (annotateCpG with cpg_b38.rda reports the percent)
- Percent of DNVs phased to the paternal chromosome of origin is consistent with published values (parent-of-origin phasing via Unfazed on the DNV calls)
- aggregated acorn input file passes readDNV schema checks: tab-delimited, sample in field 1, chr-prefixed chromosome in field 2, position, ref, alt in fields 3-5

## Limitations

- The 329,589 count is specific to the 4216 Illumina short-read WGS SSC trios; other cohorts, capture kits (WES), or sequencing platforms will yield different totals and per-individual rates
- The acorn CpG annotation file (cpg_b38.rda) is genome-build specific (b38) and too large to package with acorn, so it must be downloaded separately
- Hare v1.2 is effective for WGS and WES data only; PacBio HiFi long-read data requires the Tortoise workflow

## Evidence

- [other] Cohort total reproduces published SSC DNV count: [evidence span withheld — anchor status: not_found]
- [intro] HAT custom de novo filter on joint-genotyped files: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [abstract] Quality metrics for DNV callset: "The quality of these DNVs is high based on a series of quality metrics including number of DNVs per individual, percent of DNVs at CpG sites, and percent of DNVs phased to the paternal chromosome of"
- [readme] squirrel.py converts HAT output to acorn input: [evidence span withheld — anchor status: external_doc]
- [readme] acorn readDNV input format requirements: [evidence span withheld — anchor status: external_doc]
- [methods] Follow-up statistics on HAT output with acorn: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
