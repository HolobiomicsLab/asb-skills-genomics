---
name: minimizer-based-mapping-block-length-computation
description: 'Use when a ntLink mapping-mode PAF file of long-read to contig mappings
  (emitted with paf=True) must be assessed numerically, e.g. reproducing the Table
  5 statistics: total mappings, average block length, and average mappings per contig.'
when_to_use_negative:
- The goal is scaffolding or correction itself (Basic Protocols 1-3, or Alternate
  Protocol 2 with Tigmint-long) rather than numerically assessing the mapping output.
- The mapping file is a standard minimap2 SAM/BAM or an unlabelled PAF rather than
  ntLink's labelled PAF output, so the mlr workflow over named delimited fields does
  not apply.
- Base-level alignment quality or identity metrics are required; minimizer-based PAF
  block lengths do not encode alignment scores.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: ntLink
  role: Runs in mapping mode with paf=True to produce the labelled PAF file of long-read
    to contig mappings whose statistics are computed
  repo: https://github.com/bcgsc/ntLink
- name: miller
  role: Loads the labelled PAF as a delimited table and computes the row count, the
    mean block length from start-end coordinate fields, and per-target-contig group
    averages, writing mapping_stats.tsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:4a881bb42e748a2c565c5dfe1072353d27b5b8ad655a007d5a69443c5566f303
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - miller
  - ntLink
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# minimizer-based-mapping-block-length-computation

## Summary

Quantifies ntLink's minimizer-based long-read mappings by computing the total number of mappings, the average mapping block length from PAF start-end coordinates, and the average number of read mappings per draft contig using Miller (mlr), so that Alternate Protocol 1 mapping output can be validated against Table 5.

## When to use

Use when a ntLink mapping-mode PAF file of long-read to contig mappings (emitted with paf=True) must be assessed numerically, e.g. reproducing the Table 5 statistics: total mappings, average block length, and average mappings per contig.

## When NOT to use

- The goal is scaffolding or correction itself (Basic Protocols 1-3, or Alternate Protocol 2 with Tigmint-long) rather than numerically assessing the mapping output.
- The mapping file is a standard minimap2 SAM/BAM or an unlabelled PAF rather than ntLink's labelled PAF output, so the mlr workflow over named delimited fields does not apply.
- Base-level alignment quality or identity metrics are required; minimizer-based PAF block lengths do not encode alignment scores.

## Inputs

- Labelled ntLink PAF mapping file of long-read to contig mappings (ntLink mapping mode output with paf=True)
- Draft assembly contigs that serve as the target identifiers in the PAF records (for per-contig grouping)

## Outputs

- mapping_stats.tsv containing three statistics: total number of mappings, average mapping block length, and average read mappings per draft contig

## How to apply

Run ntLink in mapping mode with paf=True to emit the labelled read-to-contig PAF mapping file, then load it into Miller (mlr) as a delimited table. Compute the total number of mappings as the PAF row count using mlr stats/aggregation verbs (e.g. the count verb). Derive each record's mapping block length from the query/target start-end coordinate fields of the PAF record (end minus start) and take the mean across all records. Group the PAF records by target contig identifier and average the per-contig read-mapping counts to obtain the average number of read mappings per draft contig. Write the three computed statistics (total mappings, average block length, average mappings per contig) to mapping_stats.tsv and check them against the expected mapping statistics in Table 5 of the article; a mismatch indicates the wrong PAF file or mis-parsed labelled columns.

## Related tools

- **ntLink** (Runs in mapping mode with paf=True to produce the labelled PAF file of long-read to contig mappings whose statistics are computed) — https://github.com/bcgsc/ntLink
- **miller** (Loads the labelled PAF as a delimited table and computes the row count, the mean block length from start-end coordinate fields, and per-target-contig group averages, writing mapping_stats.tsv)

## Examples

```
mlr --tsv count ntlink_mappings.paf
```

## Evaluation signals

- The PAF row count (total mappings) equals the corresponding total mappings value reported in Table 5 for that assembly.
- The mean of (end - start) over PAF start-end coordinate fields reproduces the Table 5 average block length within rounding tolerance.
- Averaging per-contig mapping counts after grouping PAF records by target contig identifier reproduces the Table 5 average mappings per contig.
- mapping_stats.tsv is produced containing exactly the three expected statistics: total mappings, average block length, average mappings per contig.
- mlr parses the input without schema errors, confirming the file is the labelled ntLink PAF output generated with paf=True.

## Limitations

- Minimizer-based mappings are sketch-derived rather than base-accurate alignments, so block lengths computed from PAF coordinates are approximate extents, not alignment-verified intervals.
- The statistics are specific to the draft assembly and long-read set used in Alternate Protocol 1 and must be re-derived for other inputs; they are validated against Table 5, not against ground truth.
- The mlr aggregations depend on the labelled PAF column naming documented in the ntLink wiki; unlabelled or differently formatted PAF files will break the workflow.

## Evidence

- [abstract] Alternate Protocol 1: Generating long-read to contig mappings with ntLink: "Alternate Protocol 1: Generating long-read to contig mappings with ntLink"
- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool, and, as an example of using ntLink mappings in a different downstream application: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool, and, as an example of using ntLink mappings in a different downstream application"
- [methods] 2. Use minimizers to map the long reads to the target assembly contigs: "2. Use minimizers to map the long reads to the target assembly contigs"
- [methods] See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats.: "See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats."
