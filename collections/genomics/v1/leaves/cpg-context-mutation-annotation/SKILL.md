---
name: cpg-context-mutation-annotation
description: Use when you have a called DNV set (e.g. HAT output prepped with squirrel.py)
  and need the number of DNVs at CpG sites and their percentage as a call-quality
  metric.
when_to_use_negative:
- Your input is raw VCF trio output not yet filtered/run through the HAT de novo variant
  filter — annotate CpG only on final DNV call sets
- Your variants are INDELs or MNVs — annotateCpG extracts only SNVs, so CpG status
  will not be assigned for those variant types
- The genome build of your DNV coordinates does not match the CpG annotation file
  build (use cpg_b38.rda only for GRCh38/b38 coordinates)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: acorn
  role: R package whose annotateCpG function assigns CpG status to DNV SNVs and prints
    the number and percent at CpG sites
  repo: https://github.com/TNTurnerLab/acorn
- name: squirrel.py
  role: Converts HAT output into the tab-delimited sample/chromosome/position/ref/alt
    format required by acorn's readDNV
  repo: https://github.com/TNTurnerLab/HAT
- name: HAT
  role: Upstream workflow producing the DNV call set whose CpG fraction is annotated
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:3ea6dd38dcde8f2ab25950c5c9c5480a11c04485c562112549cd19eed486e2c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/acorn
  tools:
  - DeepVariant
  - GATK
  - GLnexus
  - HAT
  - Tortoise
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cpg-context-mutation-annotation

## Summary

Annotate de novo variants (DNVs) from a HAT/HAT-FLEX call set with whether they occur at CpG sites using the acorn R package, yielding total counts and the percent of DNVs at CpG — a key quality metric for DNV call sets.

## When to use

Use when you have a called DNV set (e.g. HAT output prepped with squirrel.py) and need the number of DNVs at CpG sites and their percentage as a call-quality metric.

## When NOT to use

- Your input is raw VCF trio output not yet filtered/run through the HAT de novo variant filter — annotate CpG only on final DNV call sets
- Your variants are INDELs or MNVs — annotateCpG extracts only SNVs, so CpG status will not be assigned for those variant types
- The genome build of your DNV coordinates does not match the CpG annotation file build (use cpg_b38.rda only for GRCh38/b38 coordinates)

## Inputs

- Tab-delimited DNV file (sample, chromosome, position, ref allele, alt allele; uncompressed or .gz) from HAT output prepped via squirrel.py
- Pre-computed CpG site annotation .rda file (e.g. cpg_b38.rda for GRCh38, downloaded from https://data.cyverse.org/dav-anon/iplant/home/tycheleturner/genomic_annotations/cpg_b38.rda)

## Outputs

- DNV object containing only SNVs with an added column noting CpG status per variant
- Reported count and percentage of DNVs at CpG sites (e.g. ~16.5% for the HG002 ~30x callset)

## How to apply

First convert HAT output into acorn input format using squirrel.py from the HAT repo, producing a tab-delimited file with sample, chromosome (with 'chr' prefix), genomic position, reference allele, and alternate allele columns. In R, load the acorn library and load the pre-computed CpG annotation file for the genome of interest (cpg_b38.rda from Cyverse for b38/GRCh38; the file is too large to ship with the package). Run annotateCpG on the DNV object, which extracts the SNVs from the DNV set, assigns CpG status to each, and prints out the number and percent of DNV SNVs at CpG. For the GIAB HG002 ~30x benchmark this yields the expected result of ~16.5% of ~1105 DNVs at CpG sites.

## Related tools

- **acorn** (R package whose annotateCpG function assigns CpG status to DNV SNVs and prints the number and percent at CpG sites) — https://github.com/TNTurnerLab/acorn
- **squirrel.py** (Converts HAT output into the tab-delimited sample/chromosome/position/ref/alt format required by acorn's readDNV) — https://github.com/TNTurnerLab/HAT
- **HAT** (Upstream workflow producing the DNV call set whose CpG fraction is annotated) — https://github.com/TNTurnerLab/HAT

## Examples

```
library('acorn'); load("cpg_b38.rda"); input <- readDNV("dnv_calls.txt"); CpGresult <- annotateCpG(DNVobject = input)
```

## Evaluation signals

- readDNV loads the file without error and chromosome strings carry the 'chr' prefix (e.g. chr1)
- annotateCpG returns only SNVs with a CpG-status column appended and prints the number of CpG sites and percent of DNV SNVs at CpG
- For the downsampled (~30x) GIAB HG002 Illumina WGS trio, the CpG fraction should approximate 16.5% of ~1105 DNVs, matching the paper's quality metric
- Chromosome coordinates in the DNV file match the build of the loaded CpG annotation (cpg_b38.rda = GRCh38)
- The function completes in at least ~1 minute for a full genome annotation (per README), so instant completion suggests the annotation file was not actually loaded

## Limitations

- The CpG annotation file is too large to package with acorn and must be downloaded separately; wrong-build annotation gives incorrect CpG calls
- annotateCpG only operates on SNVs — DNVs that are INDELs/MNVs are excluded from the CpG percentage
- The function typically takes at least one minute to run

## Evidence

- [readme] annotateCpG = Extracts single-nucleotide variants (SNVs) out from a DNV object ... and assigns whether they are at a CpG site or not: [evidence span withheld — anchor status: external_doc]
- [results] HAT called 1105 DNVs on the downsampled (~30x) HG002 ... approximately 16.5% of those DNVs found within a CpG region: [evidence span withheld — anchor status: not_found]
- [methods] To prep the output from HAT, you can run squirrel.py found here in this repo: "To prep the output from HAT, you can run squirrel.py found here in this repo."
- [readme] the chromosome data should take the form with a "chr" at the beginning (e.g., chr1): [evidence span withheld — anchor status: external_doc]
- [abstract] percent of DNVs at CpG sites ... quality metrics: "The quality of these DNVs is high based on a series of quality metrics including number of DNVs per individual, percent of DNVs at CpG sites"
