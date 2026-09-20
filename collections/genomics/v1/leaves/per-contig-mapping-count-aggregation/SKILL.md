---
name: per-contig-mapping-count-aggregation
description: Use when a labelled ntLink PAF mapping file (produced with paf=True)
  must be summarised into counts and per-contig averages, e.g. to reproduce or validate
  the mapping statistics of Table 5.
when_to_use_negative:
- The input is a raw minimizer sketch or scaffold graph rather than a completed PAF
  mapping file — the aggregation operates on PAF records only
- The goal is assembly correction or scaffolding; use Tigmint-long (Alternate Protocol
  2) or ntLink's scaffolding modes instead of just counting mappings
- Quality/identity assessment of individual alignments is needed (e.g. via minimap2
  alignment scores) — this skill only aggregates counts and coordinate-derived lengths
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: miller
  role: Delimited-data toolkit used to load the PAF file and compute row counts, averages
    of coordinate-derived block lengths, and group-by-target averages
  repo: https://github.com/johnkerl/miller
- name: ntLink
  role: Produces the PAF mapping file (long-read to contig mappings via minimizer-based
    mapping) that this skill aggregates
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:65acdb8b17833fed27e6fabee20ebc962066469418f0cf2e7dba454dc2f2e73d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/johnkerl/miller
  tools:
  - miller
  - ntLink
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# per-contig-mapping-count-aggregation

## Summary

Summarise an ntLink PAF mapping file (long-read-to-contig mappings from Alternate Protocol 1) into table-level statistics — total mappings, average mapping block length, and average mappings per draft contig — using Miller (mlr) delimited-data operations, for validation against Table 5 of the ntLink protocols paper.

## When to use

Use when a labelled ntLink PAF mapping file (produced with paf=True) must be summarised into counts and per-contig averages, e.g. to reproduce or validate the mapping statistics of Table 5.

## When NOT to use

- The input is a raw minimizer sketch or scaffold graph rather than a completed PAF mapping file — the aggregation operates on PAF records only
- The goal is assembly correction or scaffolding; use Tigmint-long (Alternate Protocol 2) or ntLink's scaffolding modes instead of just counting mappings
- Quality/identity assessment of individual alignments is needed (e.g. via minimap2 alignment scores) — this skill only aggregates counts and coordinate-derived lengths

## Inputs

- labelled ntLink PAF mapping file of long-read to query-contig mappings (ntLink mapping mode run with paf=True)

## Outputs

- mapping_stats.tsv containing total number of mappings, average mapping block length, and average number of read mappings per draft contig

## How to apply

Load the labelled PAF (tab-delimited, read-to-contig mappings output by ntLink's mapping mode) into Miller (mlr) as a delimited table. Compute the total number of mappings as the row count, and the average mapping block length from the query/target start-end coordinate fields of each PAF record. Then group the PAF records by the target contig identifier and compute the average number of read mappings per draft contig. Write the three statistics (total mappings, average block length, average mappings per contig) to mapping_stats.tsv. The rationale is that ntLink's minimizer-based mappings are lightweight enough that downstream assessment is a simple aggregation over PAF rows, and the article validates Alternate Protocol 1 by matching these computed numbers to the expected mapping statistics in Table 5.

## Related tools

- **miller** (Delimited-data toolkit used to load the PAF file and compute row counts, averages of coordinate-derived block lengths, and group-by-target averages) — https://github.com/johnkerl/miller
- **ntLink** (Produces the PAF mapping file (long-read to contig mappings via minimizer-based mapping) that this skill aggregates) — https://github.com/bcgsc/ntLink

## Examples

```
mlr --tsv --implicit-csv-header --allow-ragged-csv-input put '$block_length=$11' then stats1 -a count,mean -f block_length then nest --ivar --eol ';' -f 1 mappings.paf > mapping_stats.tsv
```

## Evaluation signals

- The computed total number of mappings equals the PAF row count and matches the total reported in Table 5 of the article
- The average mapping block length derived from the query/target start-end coordinate fields matches the Table 5 value
- The average number of read mappings per draft contig (after grouping by target contig identifier) matches the Table 5 value
- mapping_stats.tsv is written with exactly the three computed statistics in the expected schema
- Group-by aggregation covers every target contig identifier present in the PAF file with no dropped records

## Limitations

- The statistics are only as meaningful as the PAF input: they summarise minimizer-based mappings, not full read alignments, and reflect ntLink's lightweight mapping rather than an alignment-based measure
- Average mappings per contig depends on the grouping key being the target contig identifier; malformed or unlabelled PAF records will silently bias the group averages
- Validation requires the article's Table 5 expected values, which are specific to the protocol's test dataset

## Evidence

- [methods] The mapping output is assessed by counting the total number of mappings of the long reads to the query contigs.: [evidence span withheld — anchor status: not_found]
- [methods] Load the labelled ntLink PAF mapping file (read-to-contig mappings output with paf=True) into miller (mlr) as a delimited table.: [evidence span withheld — anchor status: not_found]
- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool, and, as an example of using ntLink mappings in a different downstream application"
- [readme] With Miller, you get to use named fields without needing to count positional indices... aggregate statistically: [evidence span withheld — anchor status: external_doc]
