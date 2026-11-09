---
name: sam-auxiliary-field-parsing
description: Use when a BAM/PAF produced by guppy/dorado basecalling contains a move-table
  auxiliary field (e.g. mv:B:c:5,1,1,...) or an ss tag that must be decoded into per-base
  signal matches, insertions and deletions.
when_to_use_negative:
- The alignment was produced by f5c eventalign or nanopolish signal projection rather
  than a basecaller move table — no mv tag exists to parse, so use eventalign boundaries
  directly instead
- Input is already an ss string and only alignment statistics are needed — run the
  metric sub-tool directly rather than re-encoding
- The goal is signal-to-reference plotting without any auxiliary-field manipulation
  — use squigualiser plot/plot_pileup which consume existing alignments
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Squigualiser
  role: Provides the reform sub-tool that converts the basecaller's move table to
    the f5c ss string format, and the metric sub-tool that parses ss strings to compute
    signal-alignment statistics
  repo: https://github.com/hiruna72/squigualiser
- name: f5c
  role: Defines the ss tag specification (signal-to-sequence matches, insertions,
    deletions encoded as a minimal alphanumeric string) that the encoder/decoder must
    conform to
- name: Python 3.8
  role: Implementation language for the move-table parser, ss-string encoder and ss-string
    decoder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:bc8ba56a0a0ffb497b3fbbd61ee4f3064091923e9dd3663829bd0f6c5aabf24e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - Squigualiser
  - f5c
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sam-auxiliary-field-parsing

## Summary

Parses SAM/BAM/PAF auxiliary fields — specifically the basecaller move table (mv:B:c) and the f5c-style ss tag — to recover signal-to-sequence alignments for nanopore signal visualisation and alignment-statistics computation, as used by Squigualiser's reform and metric sub-tools.

## When to use

Use when a BAM/PAF produced by guppy/dorado basecalling contains a move-table auxiliary field (e.g. mv:B:c:5,1,1,...) or an ss tag that must be decoded into per-base signal matches, insertions and deletions.

## When NOT to use

- The alignment was produced by f5c eventalign or nanopolish signal projection rather than a basecaller move table — no mv tag exists to parse, so use eventalign boundaries directly instead
- Input is already an ss string and only alignment statistics are needed — run the metric sub-tool directly rather than re-encoding
- The goal is signal-to-reference plotting without any auxiliary-field manipulation — use squigualiser plot/plot_pileup which consume existing alignments

## Inputs

- SAM/BAM file with basecaller move-table auxiliary field (mv:B:c) from guppy/dorado basecalling
- PAF file carrying signal-to-sequence alignment information
- ss string (f5c ss tag format) for decoding
- FASTQ/basecalled sequence corresponding to the move table

## Outputs

- ss string in f5c format encoding matches, insertions and deletions (PAF/TSV form, as emitted by the squigualiser reform sub-tool)
- Decoded alignment structure (matches/insertions/deletions) as parsed by the squigualiser metric sub-tool
- Alignment statistics computed from decoded ss strings
- ss_roundtrip_report.tsv tabulating encoder-decoder round-trip results

## How to apply

Parse the basecaller move table from the SAM auxiliary field (e.g. mv:B:c:5,1,1,...), taking the first integer as the stride and the remaining integers as the move array, where the number of ones equals the FASTQ sequence length. Implement the encoder that converts each move-table-derived read-signal alignment into the f5c 'ss string' format, encoding per-base matches plus insertion and deletion occurrences as a minimal alphanumeric string, emitting it in PAF/TSV form as the Squigualiser 'reform' sub-tool does. Implement the decoder that parses an ss string back into its match, insertion and deletion components, mirroring how the Squigualiser 'metric' sub-tool parses the ss string to compute alignment statistics. Squigualiser's realign method is robust to methylated bases (e.g. methylated U basecalled as C) precisely because it uses the basecaller's move table rather than pore-model signal matching.

## Related tools

- **Squigualiser** (Provides the reform sub-tool that converts the basecaller's move table to the f5c ss string format, and the metric sub-tool that parses ss strings to compute signal-alignment statistics) — https://github.com/hiruna72/squigualiser
- **f5c** (Defines the ss tag specification (signal-to-sequence matches, insertions, deletions encoded as a minimal alphanumeric string) that the encoder/decoder must conform to)
- **Python 3.8** (Implementation language for the move-table parser, ss-string encoder and ss-string decoder)

## Examples

```
mv = "5,1,1,0,1"; stride = int(mv.split(',')[0]); moves = [int(x) for x in mv.split(',')[1:]]  # parse SAM mv:B:c move table for ss-string encoding
```

## Evaluation signals

- Round-trip consistency: the ss-string decoder applied to encoder output reproduces, for every example alignment, the original move-table-derived matches/insertions/deletions exactly with no loss
- The number of ones in the move array equals the FASTQ sequence length parsed from the same record
- The first integer of the mv:B:c field is correctly treated as the stride and stripped from the move array
- Results tabulated in ss_roundtrip_report.tsv with one row per example alignment and all rows passing

## Limitations

- The move-table approach assumes the BAM/PAF carries a guppy/dorado move-table auxiliary field; alignments from f5c eventalign or nanopolish signal projection do not provide one
- Round-trip validation only guarantees structural fidelity of the encoding, not the biological accuracy of the underlying signal-to-sequence alignment
- Squigualiser has been tested with Python 3.8; parsing tooling should target compatible environments

## Evidence

- [methods] Move table to ss string conversion (reform): "`reform`:<br/>         Convert basecaller's move table to [ss string](https://hasindu2008.github.io/f5c/docs/output#ss-tag) format."
- [methods] realign uses move table: "Squigualiser's [realign method](realign.md) is robust to the methylated Us as it uses the basecaller's move table"
- [methods] metric sub-tool: "`metric`:<br/>		 A utility program to calculate some statistics of the signal alignment."
- [readme] tool purpose: "squigualiser is a tool to Visualise nanopore raw signal-base alignment."
- [other] workflow encoding/decoding: [evidence span withheld — anchor status: not_found]
