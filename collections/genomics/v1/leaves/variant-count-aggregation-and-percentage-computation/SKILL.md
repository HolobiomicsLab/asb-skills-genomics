---
name: variant-count-aggregation-and-percentage-computation
description: Use when a snpEff-annotated VCF/CSV of called variants (e.g. from a JWES
  run) must be classified by coding vs noncoding gene regions and summarized per chromosome
  with counts and percentage shares.
when_to_use_negative:
- Input is a raw (unannotated) VCF that has not yet passed through snpEff, so coding/noncoding
  classification is not yet possible
- The goal is variant calling itself (SNPs/INDELs via HaplotypeCaller) rather than
  post-calling summarization of an existing variant set
- The sample's variant set has a different scale such that per-chromosome share validation
  against JWES-reported values (e.g. 20,749 total for PRJNA207663) is not applicable
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: JWES
  role: Upstream pipeline that processes FASTQ data (FastQC QC, Trimmomatic trimming,
    BWA mapping to hg38, HaplotypeCaller variant calling) and produces the annotated
    variant set this skill aggregates
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: snpEff
  role: Annotates each called variant with interval-forest gene-region information
    used to classify variants as noncoding or protein-coding
- name: VariantsToTable
  role: Extracts called variants from the VCF into a table suitable for per-region
    and per-chromosome count aggregation
- name: JWES-DB
  role: VCF parser for JWES that can parse the annotated variant call data used in
    the aggregation
  repo: https://github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:e9a0ab86dcad85deecc754751fb0d592811c3efa985f159fa828c5a67ed06264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - JWES
  - JWES-DB
  - VariantsToTable
  - snpEff
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# variant-count-aggregation-and-percentage-computation

## Summary

Aggregates snpEff-annotated variant calls from a JWES-processed WGS sample into per-gene-region (noncoding vs protein-coding) and per-chromosome count/percentage tables, e.g. 12,979 (~63%) noncoding and 7,770 (~37%) protein-coding variants with chromosome 2 as the largest contributor.

## When to use

Use when a snpEff-annotated VCF/CSV of called variants (e.g. from a JWES run) must be classified by coding vs noncoding gene regions and summarized per chromosome with counts and percentage shares.

## When NOT to use

- Input is a raw (unannotated) VCF that has not yet passed through snpEff, so coding/noncoding classification is not yet possible
- The goal is variant calling itself (SNPs/INDELs via HaplotypeCaller) rather than post-calling summarization of an existing variant set
- The sample's variant set has a different scale such that per-chromosome share validation against JWES-reported values (e.g. 20,749 total for PRJNA207663) is not applicable

## Inputs

- snpEff-annotated VCF or CSV of called variants from a JWES pipeline run
- snpEff interval-forest gene-region annotations (noncoding vs protein-coding classification per variant)

## Outputs

- CSV table of per-chromosome variant counts and percentage shares (total and protein-coding-specific), e.g. variant_distribution_SRR891275.csv
- Summary of noncoding vs protein-coding variant counts and percentages (e.g. 12,979 ~63% / 7,770 ~37%)

## How to apply

Load the snpEff-annotated variant set (VCF/CSV) produced by the JWES pipeline (e.g. the 20,749 variants called for SRR891275, retrieved from NCBI SRA project PRJNA207663). Classify each variant as falling in a noncoding gene or a protein-coding gene based on the snpEff interval-forest annotations. Aggregate variant counts per gene region and per chromosome, computing each chromosome's share of all variants and of protein-coding variants (chromosome 2 expected as largest share, ~11.8% of all and ~7.4% of protein-coding variants). Write the combined chromosome-level and gene-region-level table with counts and percentages to a CSV such as variant_distribution_SRR891275.csv.

## Related tools

- **JWES** (Upstream pipeline that processes FASTQ data (FastQC QC, Trimmomatic trimming, BWA mapping to hg38, HaplotypeCaller variant calling) and produces the annotated variant set this skill aggregates) — https://github.com/drzeeshanahmed/JWES-DB
- **snpEff** (Annotates each called variant with interval-forest gene-region information used to classify variants as noncoding or protein-coding)
- **VariantsToTable** (Extracts called variants from the VCF into a table suitable for per-region and per-chromosome count aggregation)
- **JWES-DB** (VCF parser for JWES that can parse the annotated variant call data used in the aggregation) — https://github.com/drzeeshanahmed/JWES-DB

## Examples

```
import pandas as pd; df = pd.read_csv('SRR891275_snpeff.csv'); df.groupby(['chromosome','region']).size().to_csv('variant_distribution_SRR891275.csv')
```

## Evaluation signals

- Total variant count in the annotated table matches the JWES-reported count for the sample (20,749 variants for SRR891275 / PRJNA207663)
- Noncoding and protein-coding counts sum to the total (12,979 + 7,770 = 20,749) with percentages ~63% and ~37%
- Chromosome 2 is the largest per-chromosome contributor, holding ~11.8% of all variants and ~7.4% of protein-coding variants
- All chromosome shares sum to ~100% across the distribution table
- Output CSV contains both counts and percentages for each chromosome and gene-region class

## Limitations

- Percentages are sample-specific shares; they depend on the snpEff annotation of the hg38 reference build used by JWES and may shift with different genome builds or annotation versions
- JWES-DB provides only a minimal VCF parser ('VCF Parser for JWES') with no documented aggregation options, so tabulation logic must be implemented by the analyst
- The current JWES version offers only a command-line interface, so this aggregation step is not available through an interactive interface

## Evidence

- [results] we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
- [results] and 20 749 variants for PRJNA207663: "and 20 749 variants for PRJNA207663"
- [methods] The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38): "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [readme] # README: drzeeshanahmed/JWES-DB VCF Parser for JWES: [evidence span withheld — anchor status: not_found]
- [methods] HaplotypeCaller is used to call SNPs and INDELs: "HaplotypeCaller is used to call SNPs and INDELs"
