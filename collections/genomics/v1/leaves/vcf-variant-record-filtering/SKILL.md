---
name: vcf-variant-record-filtering
description: Use when a raw NanoSV (or other long-read SV caller) VCF must be reduced
  by dropping calls in repetitive sequence and small indels below an SV size threshold,
  with per-stage call-count validation.
when_to_use_negative:
- When the goal is to generate the initial NanoSV call set itself from BAM alignments
  rather than filter an existing VCF of SV records
- When filtering reads or alignment records (BAM/CRAM quality filtering) rather than
  variant records in a VCF
- When sub-40 bp indels must be retained, e.g. for small-variant calling, since they
  are excluded here because they typically lack split-aligning reads
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSV
  role: Produces the initial long-read SV call VCF whose records are filtered in this
    skill; its split-alignment-based calls define which small indels can be excluded
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Used upstream in the NanoSV pipeline to convert and coordinate-sort the SAM
    to BAM that NanoSV calls on, feeding the VCF being filtered
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:929ec172a096219c5aac1ef82c2479e57eff826f6b75833c33ca0208c5f83354
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# vcf-variant-record-filtering

## Summary

Post-calling filtering of a NanoSV structural-variant VCF: discard SV calls whose breakpoints fall in homopolymer/short-tandem-repeat regions, then remove sub-40 bp indels that do not typically produce split alignments, shrinking the NA12878 chromosome 1 nanopore call set from 3957 to a final 654 SVs.

## When to use

Use when a raw NanoSV (or other long-read SV caller) VCF must be reduced by dropping calls in repetitive sequence and small indels below an SV size threshold, with per-stage call-count validation.

## When NOT to use

- When the goal is to generate the initial NanoSV call set itself from BAM alignments rather than filter an existing VCF of SV records
- When filtering reads or alignment records (BAM/CRAM quality filtering) rather than variant records in a VCF
- When sub-40 bp indels must be retained, e.g. for small-variant calling, since they are excluded here because they typically lack split-aligning reads

## Inputs

- NanoSV VCF of initial SV calls (NA12878 chromosome 1 nanopore call set, 3957 records) with per-variant coordinates, type, and size
- Homopolymer and tandem-repeat (short tandem repeat) region annotations covering the reference chromosome

## Outputs

- Intermediate repeat-filtered VCF retaining 657 SV calls
- Final filtered VCF (nanosv_na12878_filtered.vcf) with 654 SV calls after removing sub-40 bp indels

## How to apply

Load the NanoSV VCF (e.g. 3957 initial SV calls from NA12878 chromosome 1 MinION nanopore data) and parse each record's coordinates, SV type, and size. Intersect SV breakpoints with homopolymer and tandem-repeat region annotations and discard every call whose breakpoint falls inside such a repeat region; retain this intermediate call set (expected 657 calls) as a VCF. From the repeat-filtered set, exclude indel calls smaller than the 40 bp SV size threshold, since sub-40 bp events do not typically result in a split alignment that NanoSV relies on, yielding the final call set of 654 SVs. Write the result to nanosv_na12878_filtered.vcf and verify row counts of both stages against the reported 657 and 654 values. The article notes this kind of extremely stringent post-calling filtering is deliberate, trading sensitivity (~75%) for precision (>95%).

## Related tools

- **NanoSV** (Produces the initial long-read SV call VCF whose records are filtered in this skill; its split-alignment-based calls define which small indels can be excluded) — https://github.com/mroosmalen/nanosv
- **sambamba** (Used upstream in the NanoSV pipeline to convert and coordinate-sort the SAM to BAM that NanoSV calls on, feeding the VCF being filtered) — https://github.com/biod/sambamba

## Evaluation signals

- Repeat-filter stage retains exactly 657 calls from the initial 3957
- Indel-size filter produces a final call set of exactly 654 calls
- No retained call in the final VCF has a breakpoint inside an annotated homopolymer or tandem-repeat region
- No retained record in the final VCF is an indel smaller than 40 bp
- Both intermediate and final outputs are valid VCFs with row counts matching the paper's filtering cascade

## Limitations

- The filtering cascade is extremely stringent; the paper reports it lowers estimated SV sensitivity to ~75% while maintaining >95% precision
- Conservative SV calling and post-calling filtering lower the fraction of novel variants relative to previously reported Pacific Biosciences data (14% vs 89%)
- Sub-40 bp indels are excluded because they do not typically result in a split alignment, so the final call set is not a complete inventory of small variants

## Evidence

- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] this is partly due to our conservative SV calling and post-calling filtering steps: "this is partly due to our conservative SV calling and post-calling filtering steps"
- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data"
- [discussion] SV calls matching any state of the art precision benchmark (>95% precision): "SV calls matching any state of the art precision benchmark (>95% precision)"
