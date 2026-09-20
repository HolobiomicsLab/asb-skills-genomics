---
name: subtelomeric-anchor-sequence-alignment
description: Use when consensus subtelomeric anchor clusters (sample.telomap.anchors.tsv
  from telomap) need to be assigned to specific T2T-CHM13 chromosome ends for chromosome-end-specific
  telomere length or TVS analyses.
when_to_use_negative:
- Subtelomeric regions are too short (e.g., after RsaI/HinfI trimming) for unambiguous
  chromosome-end identification against T2T-CHM13 — anchors will end up unmappable
- Input is raw telomeric reads rather than clustered consensus anchor sequences —
  the upstream clustering step must run first
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Performs chromosome-end mapping of subtelomeric anchor sequences and writes
    the CHROM column in sample.telomap.anchors.tsv
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: PacBio-adapted aligner (minimap2 wrapper) usable for aligning anchor sequences
    to the T2T-CHM13 reference
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:dcfbbdde63061e8b049ca6955ecd5e80fca8405b14cffad04f2a4da5127f90c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# subtelomeric-anchor-sequence-alignment

## Summary

Maps telomap's consensus subtelomeric anchor sequences to chromosome ends using the T2T-CHM13 human genome assembly, enabling chromosome-end-specific (e.g., Chr. 7q) telomere analyses per sample.

## When to use

Use when consensus subtelomeric anchor clusters (sample.telomap.anchors.tsv from telomap) need to be assigned to specific T2T-CHM13 chromosome ends for chromosome-end-specific telomere length or TVS analyses.

## When NOT to use

- Subtelomeric regions are too short (e.g., after RsaI/HinfI trimming) for unambiguous chromosome-end identification against T2T-CHM13 — anchors will end up unmappable
- Input is raw telomeric reads rather than clustered consensus anchor sequences — the upstream clustering step must run first

## Inputs

- sample.telomap.anchors.tsv anchor cluster table (BARCODE, ANCHOR_READ, ANCHOR_SEQ, READ_SUPPORT) from telomap subtelomeric clustering
- T2T-CHM13 human genome assembly reference (FASTA)

## Outputs

- Updated sample.telomap.anchors.tsv with per-cluster chromosome-end assignments (BARCODE, ANCHOR_READ, ANCHOR_SEQ, READ_SUPPORT, CHROM), including unmappable cluster IDs starting with 'U'

## How to apply

Take the anchor cluster table (sample.telomap.anchors.tsv) produced by telomap's subtelomeric clustering step, which lists ANCHOR_SEQ consensus sequences with representative reads and read-support counts. Align each consensus anchor sequence against the subtelomeric regions of the T2T-CHM13 complete telomere-to-telomere assembly to identify the unique chromosome end matching each anchor, and write the assignment into the CHROM column. Anchors whose subtelomeric region is too short or ambiguous to place uniquely are labeled with an unmappable identifier beginning with 'U'. The rationale is that grouping telomeric reads by chromosomal end yields per-nucleotide profiles for mappable ends (e.g., 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, 17p) that are unique for each patient sample.

## Related tools

- **telomap** (Performs chromosome-end mapping of subtelomeric anchor sequences and writes the CHROM column in sample.telomap.anchors.tsv) — https://github.com/cytham/telomap
- **pbmm2** (PacBio-adapted aligner (minimap2 wrapper) usable for aligning anchor sequences to the T2T-CHM13 reference) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- Every anchor cluster row in sample.telomap.anchors.tsv carries a CHROM value that is either a T2T-CHM13 chromosome end or an unmappable ID starting with 'U'
- Mappable chromosome ends are unique per patient sample and match the expected set (e.g., 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, 17p)
- Each mapped anchor retains its original BARCODE, ANCHOR_READ, ANCHOR_SEQ, and READ_SUPPORT values with only the CHROM column added
- Per-nucleotide telomere profiles can be aggregated across all reads of each assigned chromosomal end without orphan/ambiguous assignments

## Limitations

- Subtelomeric regions from HiFi reads trimmed with RsaI and HinfI can be too short for unambiguous identification of specific chromosome ends against T2T-CHM13, resulting in 'U'-labeled unmappable anchors
- Anchor-based mapping depends on subtelomeric sequence uniqueness; highly similar subtelomeres may prevent per-end resolution

## Evidence

- [methods] Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [methods] Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences: "Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences"
- [discussion] Short subtelomere limitation: "the resulting subtelomeric region from the HiFi reads could be too short for unambiguous identification of specific chromosome ends with reference to T2T-CHM13 human genome assembly"
- [readme] README workflow: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [readme] anchors output file: "sample.telomap.anchors.tsv | Table containing details of subtelomeric anchor sequences for chromosome-end mapping"
