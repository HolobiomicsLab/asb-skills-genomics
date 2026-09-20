---
name: cnv-region-reciprocal-overlap-comparison
description: 'Use when benchmarking two read-depth CNV callers (e.g. CNVnator vs CNVpytor)
  against the same BAM: you have two exported CNV call sets and need a per-region
  reciprocal overlap table plus an overall overlap fraction.'
when_to_use_negative:
- The two call sets were produced from different BAMs or different bin sizes — overlap
  would confound caller differences with input differences
- You need sensitivity/precision against truth-set CNVs rather than caller-vs-caller
  concordance; reciprocal overlap between two callers does not establish correctness
- Calls are in different genome builds and cannot be liftover-harmonized to one coordinate
  system first
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: Runs the read-depth CNV calling pipeline (-rd, -his, -partition, -call) and
    exports one call set; Python reimplementation of the CNVnator core engine
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Original read-depth CNV caller run on the same BAM with matching bin sizes
    to produce the second call set
- name: Python
  role: 'Used for the overlap computation script: harmonizing coordinates, computing
    reciprocal overlap per region, and writing the comparison TSV'
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:e9e86139921ea122da51aee2c03d761e8e7bb47818dc848d60d59a4e70c11efd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cnv-region-reciprocal-overlap-comparison

## Summary

Quantifies the agreement between CNV regions called by CNVnator and CNVpytor on the same read-depth analysis of a WGS BAM, using reciprocal overlap per region. The article reports the two callers' regions overlap by >99% despite slightly different implementations.

## When to use

Use when benchmarking two read-depth CNV callers (e.g. CNVnator vs CNVpytor) against the same BAM: you have two exported CNV call sets and need a per-region reciprocal overlap table plus an overall overlap fraction.

## When NOT to use

- The two call sets were produced from different BAMs or different bin sizes — overlap would confound caller differences with input differences
- You need sensitivity/precision against truth-set CNVs rather than caller-vs-caller concordance; reciprocal overlap between two callers does not establish correctness
- Calls are in different genome builds and cannot be liftover-harmonized to one coordinate system first

## Inputs

- Aligned WGS BAM file (e.g. Platinum Genomes CEPH trio dataset alignment)
- CNV call set exported from CNVpytor (BED/TSV region list at bin sizes 1000, 10000, 100000)
- CNV call set exported from CNVnator (same bin sizes, same BAM)
- Reference genome build identifier (GRCh37/hg19 or GRCh38) for coordinate harmonization

## Outputs

- Per-region comparison TSV with match/mismatch status and reciprocal overlap values
- Single overall CNV-region overlap fraction (expected >0.99)

## How to apply

Run the CNVpytor read-depth pipeline ('cnvpytor -root file.pytor -rd file.bam', then '-his', '-partition', and '-call' at bin sizes 1000, 10000, and 100000) and export its calls; run CNVnator's RD pipeline on the same BAM with matching bin sizes and export its calls. Harmonize both call sets to the same genome build and coordinate system (e.g. GRCh37/hg19 vs GRCh38). For each CNV region from either caller, compute reciprocal overlap and assign match/mismatch status, then calculate the overall fraction of overlapping called regions, benchmarking against the reported >99% overlap. Write the per-region comparison as TSV via 'set print_filename output.tsv' style output plus a single overlap-fraction number.

## Related tools

- **CNVpytor** (Runs the read-depth CNV calling pipeline (-rd, -his, -partition, -call) and exports one call set; Python reimplementation of the CNVnator core engine) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Original read-depth CNV caller run on the same BAM with matching bin sizes to produce the second call set)
- **Python** (Used for the overlap computation script: harmonizing coordinates, computing reciprocal overlap per region, and writing the comparison TSV) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -rd file.bam; cnvpytor -root file.pytor -his 1000 10000 100000; cnvpytor -root file.pytor -partition 1000 10000 100000; cnvpytor -root file.pytor -call 1000 10000 100000
```

## Evaluation signals

- Overall overlap fraction between the two callers' called regions exceeds 0.99, matching the article's reported result
- Every region in the comparison TSV carries a reciprocal-overlap value and a match/mismatch status; no region is left unannotated
- Both call sets derive from the same BAM and identical bin sizes (1000, 10000, 100000); bin-size metadata matches across the two pipelines
- Both call sets are in the same genome build and coordinate system after harmonization (no cross-build coordinate mixing)
- Output TSV is properly tab-delimited and parseable with matching column counts across rows

## Limitations

- CNVpytor reimplements the CNVnator engine, so high overlap (>99%) is expected by construction; it validates implementation equivalence rather than CNV-calling accuracy
- Slightly different implementations mean boundary-level discordances can occur even when the overall overlap is >99%
- Interpretation depends on matching analysis parameters; differing bin sizes or GC-correction settings would invalidate the comparison

## Evidence

- [discussion] Reported >99% overlap between CNVnator and CNVpytor called regions: "In spite of slightly different implementation, called regions by CNVnator and CNVpytor overlap by >99%."
- [intro] CNVpytor inherits the reimplemented core engine of CNVnator: "CNVpytor inherits the reimplemented core engine of CNVnator and extends visualization, modularization, performance, and functionality."
- [methods] CNVpytor read-depth calling pipeline commands: "### Call CNVs using read depth:
> cnvpytor -root file.pytor -rd file.bam"
- [readme] TSV export of annotated filtered calls via print_filename: "set print_filename output.tsv
set annotate
print calls"
