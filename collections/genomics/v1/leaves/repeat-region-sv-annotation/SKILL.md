---
name: repeat-region-sv-annotation
description: Use when a NanoSV-generated SV call set in VCF format (e.g. 36,959 or
  36,321 initial genome-wide calls, or 3957 NA12878 chr1 calls) must be reduced by
  removing breakpoints overlapping homopolymer/simple-repeat regions before downstream
  classification.
when_to_use_negative:
- Input is raw nanopore BAM/FASTQ that has not yet been called with NanoSV — SV calling
  must precede repeat-region filtering
- The goal is discovery of retrotransposon insertions or other repeat-associated novel
  SVs, which this filter deliberately discards
- The call set has already passed the repeat/homopolymer filter stage — re-running
  would double-filter and reduce counts below the reported 657/654 or 8578/6791
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Source of the SV call sets being filtered and of the ancillary repeat-filter
    configuration defining homopolymer/simple-repeat regions
  repo: mroosmalen/nanosv
- name: pysam/cyvcf2
  role: Python libraries used to load the NanoSV VCF, parse variant coordinates/type/size,
    and write the filtered VCF
- name: sambamba
  role: Pre-processing tool (view/sort) for producing the coordinate-sorted BAMs that
    NanoSV requires upstream of this filter
  repo: biod/sambamba
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:f71ec97d79cfaf2cfed0874cad33e7eb4818d639f477e92543234ce5c431bd50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - pysam/cyvcf2
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Repeat-region SV annotation and filtering

## Summary

Intersects NanoSV structural-variant breakpoint positions with homopolymer and simple/tandem-repeat region annotations to discard SV calls whose breakpoints fall inside such repeats, producing the intermediate call sets (e.g. 8578 and 6791 SVs for Patient 1 and 2) that precede random-forest high-confidence filtering.

## When to use

Use when a NanoSV-generated SV call set in VCF format (e.g. 36,959 or 36,321 initial genome-wide calls, or 3957 NA12878 chr1 calls) must be reduced by removing breakpoints overlapping homopolymer/simple-repeat regions before downstream classification.

## When NOT to use

- Input is raw nanopore BAM/FASTQ that has not yet been called with NanoSV — SV calling must precede repeat-region filtering
- The goal is discovery of retrotransposon insertions or other repeat-associated novel SVs, which this filter deliberately discards
- The call set has already passed the repeat/homopolymer filter stage — re-running would double-filter and reduce counts below the reported 657/654 or 8578/6791

## Inputs

- NanoSV call set in VCF format (e.g. nanosv_na12878.vcf with 3957 SVs, or genome-wide patient VCFs with 36,959/36,321 SVs)
- Homopolymer and simple-repeat/tandem-repeat region annotations (repeat-filter configuration and ancillary BED files distributed with NanoSV)

## Outputs

- Intermediate repeat-filtered VCF (expected 657 calls for NA12878 chr1; 8578 and 6791 calls for Patient 1 and Patient 2)
- Final filtered SV call set VCF (e.g. nanosv_na12878_filtered.vcf with 654 calls after the <40bp indel filter)
- Row-count validation report for each filter stage

## How to apply

Load the NanoSV output VCF with pysam/cyvcf2 and parse each variant's coordinates, type, and size. Intersect SV breakpoints with homopolymer and tandem-repeat annotations taken from the repeat-filter configuration distributed with NanoSV, discarding every call whose breakpoint falls within a repeat region; keep the repeat-filtered call set as an intermediate VCF. On NA12878 chromosome 1 this cascade reduces 3957 initial calls to 657 (repeat filter) and then 654 after excluding sub-40bp indels that rarely produce split alignments; on the patient genomes it reduces 36,959/36,321 calls to 8578/6791. Retain only records whose breakpoints lie outside repeat intervals, write a valid SV VCF, and verify intermediate row counts against the reported values before applying the NA12878-trained random forest (which yields 3271/3345 final calls).

## Related tools

- **NanoSV** (Source of the SV call sets being filtered and of the ancillary repeat-filter configuration defining homopolymer/simple-repeat regions) — mroosmalen/nanosv
- **pysam/cyvcf2** (Python libraries used to load the NanoSV VCF, parse variant coordinates/type/size, and write the filtered VCF)
- **sambamba** (Pre-processing tool (view/sort) for producing the coordinate-sorted BAMs that NanoSV requires upstream of this filter) — biod/sambamba

## Examples

```
python -c "import pysam; vcf=pysam.VariantFile('nanosv_na12878.vcf'); n=sum(1 for rec in vcf.fetch() if rec.stop not in repeat_intervals); print('retained:', n)"
```

## Evaluation signals

- Repeat-filter stage retains exactly 657 calls from the initial 3957 NA12878 chr1 calls
- Indel-size filter stage (<40bp) produces exactly 654 final NA12878 chr1 calls
- Patient repeat-filtered intermediate counts equal 8578 (Patient 1) and 6791 (Patient 2)
- Every output record is a valid VCF structural-variant entry with parseable coordinates, type, and size
- Final high-confidence sets after the subsequent NA12878-trained random forest equal 3271 (Patient 1) and 3345 (Patient 2)

## Limitations

- Filtering is extremely stringent by design: the paper reports ~75% estimated sensitivity at >95% precision, so genuine SVs overlapping repeats are lost
- Discarding repeat-overlapping breakpoints conflicts with the goal of cataloguing retrotransposon insertions, a large proportion of the novel inherited variants the paper highlights
- Complex repeat regions and segmental duplications remain an unresolved challenge for building larger SV catalogs in this pipeline

## Evidence

- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data"
- [discussion] Stringent filtering lowers sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] Conservative post-calling filtering: "this is partly due to our conservative SV calling and post-calling filtering steps"
- [abstract] Novel retrotransposon insertions are recovered by genome-wide SV surveillance: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
