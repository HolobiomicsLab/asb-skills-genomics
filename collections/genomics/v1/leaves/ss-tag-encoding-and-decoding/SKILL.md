---
name: ss-tag-encoding-and-decoding
description: Use when a workflow must convert basecaller move tables (e.g. mv:B:c
  auxiliary fields in SAM/BAM) into compact ss-string signal alignments, or parse
  such ss strings back into matches/insertions/deletions for nanopore signal-alignment
  statistics.
when_to_use_negative:
- Signal-to-reference alignments are already produced by f5c eventalign or nanopolish
  with their own event-alignment output — use those alignments directly rather than
  re-encoding from move tables
- Input has no basecaller move table (e.g. signal alignment via nanopolish signal
  projection or squigualiser signal simulation without moves)
- The goal is visualisation only (plot, plot_pileup, plot_tracks) rather than alignment
  encoding or statistic computation
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Squigualiser
  role: Provides the 'reform' sub-tool that converts the basecaller's move table to
    the f5c ss string format, and the 'metric' sub-tool that parses ss strings to
    compute signal-alignment statistics
  repo: https://github.com/hiruna72/squigualiser
- name: f5c
  role: Defines the ss tag output format specification that the encoder/decoder must
    conform to
- name: Python 3.8
  role: Language used to implement and run the encoder/decoder round-trip validation
    (squigualiser is developed in Python 3.8)
  repo: https://github.com/hiruna72/squigualiser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:422043cc9a9015e1de4e47ca6cfba372cdecd09c28f73809a94750617f0a7bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - Squigualiser
  - f5c
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ss-tag-encoding-and-decoding

## Summary

Encodes and decodes nanopore signal-to-sequence alignments as the f5c-style 'ss' auxiliary tag (the move-table-to-ss-string 'reform' conversion in Squigualiser), enabling signal alignments to be carried in BAM/PAF files and parsed for alignment statistics via the 'metric' sub-tool.

## When to use

Use when a workflow must convert basecaller move tables (e.g. mv:B:c auxiliary fields in SAM/BAM) into compact ss-string signal alignments, or parse such ss strings back into matches/insertions/deletions for nanopore signal-alignment statistics.

## When NOT to use

- Signal-to-reference alignments are already produced by f5c eventalign or nanopolish with their own event-alignment output — use those alignments directly rather than re-encoding from move tables
- Input has no basecaller move table (e.g. signal alignment via nanopolish signal projection or squigualiser signal simulation without moves)
- The goal is visualisation only (plot, plot_pileup, plot_tracks) rather than alignment encoding or statistic computation

## Inputs

- SAM/BAM file containing basecaller move table auxiliary field (e.g. mv:B:c:5,1,1,...)
- Basecalled FASTQ/FASTA sequence for the read
- Existing ss strings in PAF or BAM auxiliary tags (for decoding)

## Outputs

- ss string in PAF/TSV form (as produced by the squigualiser reform sub-tool)
- Decoded match/insertion/deletion alignment structure per read
- ss_roundtrip_report.tsv tabulating round-trip consistency

## How to apply

Parse the basecaller move table from the SAM auxiliary field (e.g. mv:B:c:5,1,1,...), taking the first integer as the stride and the remaining integers as the move array, where the count of ones equals the FASTQ sequence length. Run Squigualiser's 'reform' sub-tool logic to convert each move-table-derived read-signal alignment into the f5c 'ss string' format — an auxiliary tag inspired by the CIGAR string that encodes per-base matches plus insertion and deletion occurrences as a minimal alphanumeric string — and emit it in PAF/TSV form. Implement the mirror decoder that parses an ss string back into its match, insertion and deletion components, as Squigualiser's 'metric' sub-tool does when computing alignment statistics. Validate by encoder-then-decoder round trips on example signal alignments, tabulating results in ss_roundtrip_report.tsv.

## Related tools

- **Squigualiser** (Provides the 'reform' sub-tool that converts the basecaller's move table to the f5c ss string format, and the 'metric' sub-tool that parses ss strings to compute signal-alignment statistics) — https://github.com/hiruna72/squigualiser
- **f5c** (Defines the ss tag output format specification that the encoder/decoder must conform to)
- **Python 3.8** (Language used to implement and run the encoder/decoder round-trip validation (squigualiser is developed in Python 3.8)) — https://github.com/hiruna72/squigualiser

## Evaluation signals

- Encoder-then-decoder round trip: for every example alignment, decoded matches/insertions/deletions exactly reproduce the original move-table alignment with no loss
- Number of ones in the parsed move array equals the FASTQ sequence length
- First integer of the mv:B:c auxiliary field is correctly consumed as the stride, not as a move
- Emitted ss strings conform to the f5c ss tag format referenced at hasindu2008.github.io/f5c/docs/output#ss-tag
- All validation rows tabulated in ss_roundtrip_report.tsv pass round-trip consistency

## Limitations

- Encoding requires a basecaller move table; alignment methods without moves (e.g. nanopolish signal projection) cannot be re-encoded via this route
- Squigualiser's realign uses the move table specifically because it is robust to methylated Us, which the basecaller almost always calls as C — ss encoding inherits this move-table provenance
- The ss tag is an auxiliary, minimal alphanumeric encoding; consumers must support the f5c ss-tag convention to read it from BAM/PAF

## Evidence

- [methods] reform converts move table to ss string: "`reform`:<br/>         Convert basecaller's move table to [ss string](https://hasindu2008.github.io/f5c/docs/output#ss-tag) format."
- [methods] metric parses ss strings for statistics: "`metric`:<br/>		 A utility program to calculate some statistics of the signal alignment."
- [other] ss tag encodes signal-to-sequence alignment: [evidence span withheld — anchor status: not_found]
- [methods] realign uses move table, methylated-U robust: "Squigualiser's [realign method](realign.md) is robust to the methylated Us as it uses the basecaller's move table"
