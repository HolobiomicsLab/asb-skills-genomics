---
name: miller-command-line-data-aggregation
description: Use when a ntLink mapping run (ntLink map with paf=True) has produced
  a labelled PAF file of long-read-to-contig mappings and you must reproduce the mapping
  statistics of Table 5 (total mappings, average block length, average mappings per
  contig).
when_to_use_negative:
- The file is not a PAF mapping output from ntLink's mapping mode (e.g. it is a scaffolded
  assembly FASTA or a Tigmint-long correction report) — these statistics are defined
  only over read-to-contig mapping records
- You need alignment-quality assessment of long-read mappings (identity, MAPQ, misassemblies)
  rather than counts and block-length summaries; the article directs those use cases
  to other protocols (e.g. Tigmint-long correction) or tools like QUAST
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: miller
  role: Command-line tool used to load the PAF file as a delimited table and compute
    row counts, block-length averages, and per-target-contig grouped averages
  repo: https://github.com/johnkerl/miller
- name: ntLink
  role: Generates the labelled PAF read-to-contig mapping file (mapping mode, paf=True)
    that is the input to the Miller aggregation
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:b6f2f1d9f07aa88b57ebd7c25462cf15d77dc2d311aee2a36f555e33c6199d27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/johnkerl/miller
  tools:
  - miller
  - ntLink
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# miller-command-line-data-aggregation

## Summary

Use Miller (mlr), the awk/sed-like tool for delimited data, to compute summary statistics from ntLink's labelled PAF read-to-contig mapping file — total mappings, average mapping block length, and average mappings per draft contig — as the quality assessment of Alternate Protocol 1.

## When to use

Use when a ntLink mapping run (ntLink map with paf=True) has produced a labelled PAF file of long-read-to-contig mappings and you must reproduce the mapping statistics of Table 5 (total mappings, average block length, average mappings per contig).

## When NOT to use

- The file is not a PAF mapping output from ntLink's mapping mode (e.g. it is a scaffolded assembly FASTA or a Tigmint-long correction report) — these statistics are defined only over read-to-contig mapping records
- You need alignment-quality assessment of long-read mappings (identity, MAPQ, misassemblies) rather than counts and block-length summaries; the article directs those use cases to other protocols (e.g. Tigmint-long correction) or tools like QUAST

## Inputs

- Labelled ntLink PAF mapping file (long-read to query-contig mappings output by ntLink mapping mode with paf=True)

## Outputs

- mapping_stats.tsv containing total number of mappings, average mapping block length, and average number of read mappings per draft contig

## How to apply

Load the labelled PAF file produced by ntLink's mapping mode into Miller as a tab-delimited table (PAF has no header, so use indexed/positional field access). Use Miller stats/aggregation verbs to first compute the total number of mappings as the record row count, then derive the mapping block length from the query/target start-end coordinate fields of each PAF record and average it. Finally group the PAF records by the target contig identifier field with Miller grouping and compute the average number of read mappings per draft contig. Write the three statistics (total mappings, average block length, average mappings per contig) to mapping_stats.tsv. This step assesses the mapping output by counting the total number of mappings of the long reads to the query contigs, per the article's Alternate Protocol 1.

## Related tools

- **miller** (Command-line tool used to load the PAF file as a delimited table and compute row counts, block-length averages, and per-target-contig grouped averages) — https://github.com/johnkerl/miller
- **ntLink** (Generates the labelled PAF read-to-contig mapping file (mapping mode, paf=True) that is the input to the Miller aggregation) — https://github.com/bcgsc/ntLink

## Examples

```
mlr --nidx --fs '\t' put '$block=$9-$8' then stats1 -a count,mean -f block -g 6 then stats count ntlink_mappings.paf > mapping_stats.tsv
```

## Evaluation signals

- The computed total number of mappings equals the row count of the PAF file and matches the corresponding value in Table 5 of the article
- The average mapping block length, derived from the PAF query/target start-end coordinates, matches the expected value reported in Table 5
- The average number of read mappings per draft contig, computed after grouping PAF records by target contig identifier, matches the Table 5 value
- mapping_stats.tsv contains exactly the three statistics (total mappings, average block length, average mappings per contig) in a well-formed TSV

## Limitations

- Validation is by exact match to Table 5 of the article; deviations indicate the mapping run parameters or the Miller field selection do not match the protocol
- PAF is a header-less tab-delimited format, so Miller must be invoked with positional/indexed field access rather than named fields unless the file has been relabelled; see the ntLink wiki for output file formats

## Evidence

- [abstract] Generating long-read to contig mappings with ntLink (Alternate Protocol 1): [evidence span withheld — anchor status: not_found]
- [abstract] mappings for downstream applications: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [readme] Miller description: [evidence span withheld — anchor status: external_doc]
- [readme] Miller aggregation capability: [evidence span withheld — anchor status: external_doc]
- [methods] ntLink output formats: "See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats."
