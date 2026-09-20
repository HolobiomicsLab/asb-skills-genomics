---
name: snpeff-annotation-parsing
description: Use when the input is a snpEff-annotated VCF/CSV from the JWES pipeline
  and the goal is to quantify coding vs. noncoding variant burden and per-chromosome
  variant distribution.
when_to_use_negative:
- Input is an unannotated VCF that has not yet been run through snpEff — there are
  no interval-forest annotations to classify variants against
- The goal is QC, trimming, alignment, or variant calling on raw FASTQ reads rather
  than interpreting already-annotated variant calls (use the JWES FastQC/Trimmomatic/BWA
  stages instead)
tools:
- name: snpEff
  role: Annotates called variants with interval-forest gene-region (coding/noncoding)
    annotations that this skill parses from the VCF
- name: VariantsToTable
  role: Extracts selected fields (chromosome, position, annotations) from the snpEff-annotated
    VCF into a flat table for aggregation
- name: JWES
  role: Pipeline whose annotated variant output (VCF/CSV) is the input; JWES-DB provides
    a VCF parser for JWES output
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:f5cfb133c6aff18e0ef99ba9ee2506b6c57a7e2d42a1cf5ff5e1f986b0d3a7f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - VariantsToTable
  - snpEff
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# snpEff annotation parsing

## Summary

Parses the snpEff-annotated variant set (VCF/CSV) emitted by the JWES pipeline to classify each variant as falling in a noncoding vs. protein-coding gene and to tabulate per-chromosome variant counts and shares.

## When to use

Use when the input is a snpEff-annotated VCF/CSV from the JWES pipeline and the goal is to quantify coding vs. noncoding variant burden and per-chromosome variant distribution.

## When NOT to use

- Input is an unannotated VCF that has not yet been run through snpEff — there are no interval-forest annotations to classify variants against
- The goal is QC, trimming, alignment, or variant calling on raw FASTQ reads rather than interpreting already-annotated variant calls (use the JWES FastQC/Trimmomatic/BWA stages instead)

## Inputs

- snpEff-annotated VCF (or CSV) produced by the JWES pipeline for a WGS/WES sample
- snpEff interval-forest gene annotations used to tag coding vs. noncoding variants
- human reference genome build hg38 used during alignment and annotation

## Outputs

- Gene-region distribution table (noncoding vs. protein-coding variant counts and percentages)
- Per-chromosome distribution table with per-chromosome counts, share of all variants, and share of protein-coding variants
- Combined CSV (e.g., variant_distribution_SRR891275.csv)

## How to apply

Load the snpEff-annotated variant set (VCF/CSV) for the JWES-processed sample (e.g., SRR891275 from PRJNA207663, which carried 20,749 variants). Classify each variant as falling in a noncoding gene or a protein-coding gene based on the snpEff interval-forest annotations carried in the VCF annotation fields. Aggregate variant counts per gene region and per chromosome, then compute each chromosome's share of all variants and of protein-coding variants. Write the combined chromosome-level and gene-region-level distribution table with counts and percentages (e.g., to variant_distribution_SRR891275.csv). Extraction of table fields from the VCF can be done with GATK's VariantsToTable, and the JWES-DB repo provides a VCF parser for JWES output.

## Related tools

- **snpEff** (Annotates called variants with interval-forest gene-region (coding/noncoding) annotations that this skill parses from the VCF)
- **VariantsToTable** (Extracts selected fields (chromosome, position, annotations) from the snpEff-annotated VCF into a flat table for aggregation)
- **JWES** (Pipeline whose annotated variant output (VCF/CSV) is the input; JWES-DB provides a VCF parser for JWES output) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
gatk VariantsToTable -V SRR891275.snpeff.vcf -F CHROM -F POS -F REF -F ALT -F ANN -O SRR891275.table
```

## Evaluation signals

- Total classified variants must equal the annotated variant set size (for SRR891275: 12,979 noncoding + 7,770 protein-coding = 20,749)
- Region shares must reproduce ~63% noncoding and ~37% protein-coding
- Chromosome 2 must emerge as the largest contributor (~11.8% of all variants and ~7.4% of protein-coding variants)
- All per-chromosome percentage shares must sum to 100% within rounding error
- Every variant row in the output table must carry a coding/noncoding classification; unclassified rows indicate malformed or missing snpEff ANN fields

## Limitations

- The JWES-DB README describes the tool only minimally ('VCF Parser for JWES'), so parsing behavior must be validated against the reported reference counts rather than documented options
- Classification quality depends entirely on the snpEff interval-forest annotations; variants outside annotated gene regions cannot be binned as coding/noncoding
- The reported distributions are specific to the JWES pipeline, hg38-based annotation, and sample SRR891275 (PRJNA207663); other samples/projects will have different counts

## Evidence

- [other] snpEff annotated variant set: [evidence span withheld — anchor status: not_found]
- [other] coding vs noncoding classification: [evidence span withheld — anchor status: not_found]
- [results] variant counts for sample: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
- [results] total variants PRJNA207663: "and 20 749 variants for PRJNA207663"
