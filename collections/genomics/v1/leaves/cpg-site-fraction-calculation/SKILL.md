---
name: cpg-site-fraction-calculation
description: Use when you have a filtered DNV callset (e.g. from HAT on GIAB HG002
  PacBio HiFi replicates) and need the percentage of DNVs at CpG sites as a call-quality
  metric or replicate-comparison statistic.
when_to_use_negative:
- The callset contains only INDELs or MNVs — annotateCpG extracts only SNVs, so CpG
  fractions would be undefined or misleading
- The reference genome build is not b38 and no matching pre-computed CpG rda file
  is available
- Variants are already annotated with CpG status by another tool and you only need
  per-sample summaries rather than re-annotation
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: acorn
  role: R package providing readDNV and annotateCpG, which assigns CpG status to DNV
    SNVs and prints the CpG count and percent
  repo: https://github.com/TNTurnerLab/acorn
- name: HAT (squirrel.py)
  role: Converts HAT de novo variant output into the tab-delimited sample/chr/pos/ref/alt
    format that acorn's readDNV requires
  repo: https://github.com/TNTurnerLab/HAT
- name: Tortoise
  role: Produces the PacBio HiFi-supported DNV callsets whose CpG fractions this skill
    summarizes
  repo: https://github.com/TNTurnerLab/Tortoise
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:85ad1c887b6fcdf4edc11915beea85cbc9f9a6c6bee2bc27eba1c34054a39fc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/acorn
  tools:
  - DeepVariant
  - GLnexus
  - HAT (squirrel.py)
  - Tortoise
  - acorn
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cpg-site-fraction-calculation

## Summary

Computes the fraction of de novo variant (DNV) SNVs that occur at CpG sites, a standard DNV call-quality metric in the HAT workflow, using the acorn R package's annotateCpG function against a pre-computed CpG annotation file (e.g. cpg_b38.rda).

## When to use

Use when you have a filtered DNV callset (e.g. from HAT on GIAB HG002 PacBio HiFi replicates) and need the percentage of DNVs at CpG sites as a call-quality metric or replicate-comparison statistic.

## When NOT to use

- The callset contains only INDELs or MNVs — annotateCpG extracts only SNVs, so CpG fractions would be undefined or misleading
- The reference genome build is not b38 and no matching pre-computed CpG rda file is available
- Variants are already annotated with CpG status by another tool and you only need per-sample summaries rather than re-annotation

## Inputs

- Filtered DNV callset from HAT (tab-delimited file: sample, chromosome, position, reference allele, alternate allele; plain or .gz compressed), prepped via squirrel.py
- Pre-computed CpG site annotation file for the reference genome build, e.g. cpg_b38.rda for GRCh38

## Outputs

- DNV object annotated with a column indicating whether each SNV is at a CpG site
- Console-reported counts: number of DNVs at CpG and percent of DNV SNVs at CpG (expected ~16.6% and ~17.0% for the two HG002 PacBio HiFi replicates)

## How to apply

After HAT (Tortoise branch for PacBio HiFi) and its custom de novo variant filter yield a per-replicate DNV callset, convert HAT output to acorn input using squirrel.py. In R, load the acorn library and read the DNV file with readDNV (tab-delimited: sample, chromosome with 'chr' prefix, position, ref, alt), then load a pre-computed CpG annotation rda matching your reference build (b38 available from the Turner Lab Cyverse link). Run annotateCpG, which subsets to SNVs, flags each DNV as CpG or not, and prints both the number of CpG DNVs and the percent of DNV SNVs at CpG. Judge the run against reported baselines: replicate 1 of HG002 PacBio HiFi yielded 1108 DNVs with ~16.6% at CpG, replicate 2 yielded 1106 DNVs with ~17.0% at CpG.

## Related tools

- **acorn** (R package providing readDNV and annotateCpG, which assigns CpG status to DNV SNVs and prints the CpG count and percent) — https://github.com/TNTurnerLab/acorn
- **HAT (squirrel.py)** (Converts HAT de novo variant output into the tab-delimited sample/chr/pos/ref/alt format that acorn's readDNV requires) — https://github.com/TNTurnerLab/HAT
- **Tortoise** (Produces the PacBio HiFi-supported DNV callsets whose CpG fractions this skill summarizes) — https://github.com/TNTurnerLab/Tortoise

## Examples

```
library('acorn'); load('cpg_b38.rda'); input <- readDNV('hat_dnv_output.txt.gz'); CpGresult <- annotateCpG(DNVobject = input)
```

## Evaluation signals

- Each HG002 PacBio HiFi replicate yields ~1108 and ~1106 DNVs before CpG summarization, matching the reported callset sizes
- Percent of DNVs at CpG falls near ~16.6% (replicate 1) and ~17.0% (replicate 2)
- annotateCpG prints both the number of CpG DNVs and the percent of DNV SNVs at CpG, so both values are reproducible from the console output
- Only SNVs contribute to the CpG fraction; verify INDELs/MNVs were excluded via extractSNVs behavior before computing percentages
- Input file passes readDNV parsing: fields in order sample, chromosome (with 'chr' prefix), position, reference allele, alternate allele, tab-delimited

## Limitations

- annotateCpG only operates on SNVs; the CpG fraction metric excludes INDELs and MNVs by design
- A build-specific pre-computed CpG rda file is required (e.g. cpg_b38.rda); it is too large to ship inside the acorn package and must be downloaded separately
- The annotateCpG function typically takes at least one minute to run

## Evidence

- [abstract] The quality of these DNVs is high based on a series of quality metrics including number of DNVs per individual, percent of DNVs at CpG sites: "The quality of these DNVs is high based on a series of quality metrics including number of DNVs per individual, percent of DNVs at CpG sites"
- [methods] If you would like to do some followup statistics on the output from HAT, you can install the R package acorn: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
- [readme] This function also prints out the number of CpG and the percent of DNV SNVs at CpG: [evidence span withheld — anchor status: external_doc]
- [results] HAT called 1108 DNVs from one HG002 PacBio HiFi replicate and 1106 from the second, with ∼16.6% and ∼17.0% of DNVs found at CpG sites, respectively.: [evidence span withheld — anchor status: not_found]
