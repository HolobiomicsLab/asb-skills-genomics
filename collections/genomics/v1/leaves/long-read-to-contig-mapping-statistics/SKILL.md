---
name: long-read-to-contig-mapping-statistics
description: Use when a labelled ntLink PAF mapping file (read-to-contig mappings
  generated with paf=True) must be summarized to validate long-read mappings against
  expected statistics from the ntLink article's Table 5.
when_to_use_negative:
- The goal is assembly quality assessment (N50, misassemblies) of a scaffolded assembly
  rather than summarizing mapping records — use QUAST instead
- The mappings were produced by an aligner other than ntLink's mapping mode (e.g.
  minimap2) and the expected Table 5 statistics refer specifically to ntLink minimizer-based
  mappings
- The file is an unlabelled scaffold graph or FASTA from Basic Protocols 1–3 rather
  than a PAF mapping output
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: miller
  role: Delimited-data toolkit used to parse the PAF file, aggregate row counts, compute
    average block lengths from coordinate fields, and group by target contig
  repo: https://github.com/johnkerl/miller
- name: ntLink
  role: Generates the labelled PAF read-to-contig mappings in its mapping mode (paf=True)
    that are the input to this statistics step
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:e626ad1243f1808770a06dee0cf0dcbab39c8e56d6f2c9dd868d8fe75daa32ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/johnkerl/miller
  tools:
  - miller
  - ntLink
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# long-read-to-contig-mapping-statistics

## Summary

Assesses the PAF mapping file produced by ntLink's mapping mode (Alternate Protocol 1) by computing total read-to-contig mappings, average mapping block length, and average mappings per draft contig using Miller (mlr), as reported in Table 5 of the ntLink protocol.

## When to use

Use when a labelled ntLink PAF mapping file (read-to-contig mappings generated with paf=True) must be summarized to validate long-read mappings against expected statistics from the ntLink article's Table 5.

## When NOT to use

- The goal is assembly quality assessment (N50, misassemblies) of a scaffolded assembly rather than summarizing mapping records — use QUAST instead
- The mappings were produced by an aligner other than ntLink's mapping mode (e.g. minimap2) and the expected Table 5 statistics refer specifically to ntLink minimizer-based mappings
- The file is an unlabelled scaffold graph or FASTA from Basic Protocols 1–3 rather than a PAF mapping output

## Inputs

- Labelled ntLink PAF mapping file of long-read to query-contig mappings (produced by ntLink mapping mode with paf=True)

## Outputs

- mapping_stats.tsv containing three statistics: total number of mappings, average mapping block length, and average number of read mappings per draft contig

## How to apply

Load the labelled ntLink PAF file (the read-to-contig mappings output when ntLink mapping mode is run with paf=True) into Miller (mlr) as a delimited table, since PAF records are tab-separated with named query/target start-end coordinate fields. Use mlr stats/aggregation operations to compute the row count (total mappings) and the average mapping block length derived from the query/target start-end coordinates of each PAF record. Group the PAF records by target contig identifier with mlr to compute the average number of read mappings per draft contig. Write the three computed statistics — total mappings, average block length, average mappings per contig — to mapping_stats.tsv. Validation is by matching these statistics to the corresponding expected values reported in Table 5 of the article; the wiki on the BirolLab/ntLink repository documents the output file formats.

## Related tools

- **miller** (Delimited-data toolkit used to parse the PAF file, aggregate row counts, compute average block lengths from coordinate fields, and group by target contig) — https://github.com/johnkerl/miller
- **ntLink** (Generates the labelled PAF read-to-contig mappings in its mapping mode (paf=True) that are the input to this statistics step) — https://github.com/bcgsc/ntLink

## Examples

```
mlr --tsv stats1 -a count,mean -f length then stats1 -a mean -f n_mappings -g target ~/.ntlink_mappings.paf > mapping_stats.tsv
```

## Evaluation signals

- Total number of mappings equals the PAF row count and matches the corresponding value in Table 5 of the article
- Average mapping block length computed from query/target start-end coordinate fields matches the Table 5 expectation
- Average number of read mappings per draft contig, computed by grouping PAF records on target contig identifier, matches Table 5
- Output file mapping_stats.tsv is produced with exactly the three computed statistics
- PAF records are well-formed: each row carries query and target start-end coordinates usable for block-length calculation

## Limitations

- The statistics are meaningful only for ntLink minimizer-based mappings (paf=True output), not for full read alignments from other tools
- Validation depends on comparison against the article's Table 5 expected values; other datasets will not match those reference numbers
- Miller must parse the PAF as a delimited table, so unusual or malformed PAF records could skew the averages

## Evidence

- [abstract] Alternate Protocol 1: Generating long-read to contig mappings with ntLink: "Alternate Protocol 1: Generating long-read to contig mappings with ntLink"
- [abstract] the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool, and, as an example of using ntLink mappings in a different downstream application"
- [readme] See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats.: "See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats."
