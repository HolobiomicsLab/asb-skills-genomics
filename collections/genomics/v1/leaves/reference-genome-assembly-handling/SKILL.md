---
name: reference-genome-assembly-handling
description: Use when consensus subtelomeric anchor sequences (e.g., from a sample.telomap.anchors.tsv
  cluster table) must be assigned to unique chromosomal ends against the T2T-CHM13
  reference for chromosome-end-specific telomere analysis.
when_to_use_negative:
- Subtelomeric regions are too short for unambiguous chromosome-end identification
  (e.g., after RsaI/HinfI trimming), so most anchors would be labeled 'U'
- Input reads are not yet clustered into consensus subtelomeric anchors — run the
  subtelomeric clustering step first
- Using a non-telomere-to-telomere reference lacking complete subtelomeric sequence,
  since chromosome-end mapping is based on the T2T-CHM13 assembly
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Identifies telomeric reads, clusters subtelomeric regions into consensus anchor
    sequences, and maps those anchors to chromosome ends based on the T2T-CHM13 assembly
  repo: https://github.com/cytham/telomap
- name: T2T-CHM13
  role: Telomere-to-telomere human reference assembly providing complete subtelomeric
    sequence used as the mapping target for chromosome-end assignment
- name: pbmm2
  role: Minimap2-based PacBio aligner available for aligning long reads to the reference
    assembly
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:85c43f2616da22ffba5decaecf01aec630dce1b3b16fff05603db951d99e7dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - T2T-CHM13
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# reference-genome-assembly-handling

## Summary

Assigns consensus subtelomeric anchor sequences to specific chromosome ends by aligning them against the T2T-CHM13 telomere-to-telomere human assembly, enabling chromosome-end-specific telomere analyses (e.g., Chr. 7q) in the telomap workflow.

## When to use

Use when consensus subtelomeric anchor sequences (e.g., from a sample.telomap.anchors.tsv cluster table) must be assigned to unique chromosomal ends against the T2T-CHM13 reference for chromosome-end-specific telomere analysis.

## When NOT to use

- Subtelomeric regions are too short for unambiguous chromosome-end identification (e.g., after RsaI/HinfI trimming), so most anchors would be labeled 'U'
- Input reads are not yet clustered into consensus subtelomeric anchors — run the subtelomeric clustering step first
- Using a non-telomere-to-telomere reference lacking complete subtelomeric sequence, since chromosome-end mapping is based on the T2T-CHM13 assembly

## Inputs

- Anchor cluster table sample.telomap.anchors.tsv with ANCHOR_SEQ, ANCHOR_READ, and READ_SUPPORT columns (from telomap subtelomeric clustering)
- T2T-CHM13 human genome assembly reference (FASTA)

## Outputs

- Updated sample.telomap.anchors.tsv where each cluster carries a CHROM value that is a T2T-CHM13 chromosome end (e.g., 7q, 2p, 11q) or an unmappable cluster ID starting with 'U'
- Per-chromosome-end grouping of telomeric reads enabling end-specific per-nucleotide telomere length profiles

## How to apply

Start from the anchor cluster table (sample.telomap.anchors.tsv) produced by subtelomeric read clustering, which holds ANCHOR_SEQ consensus sequences with representative reads and read-support counts. Align each consensus anchor sequence to the subtelomeric regions of the T2T-CHM13 human genome assembly using telomap's chromosome-end mapping module to identify the unique chromosome end matching each anchor. Assign each anchor cluster its mapped chromosome end in the CHROM column; anchors that cannot be placed unambiguously are labeled with an unmappable identifier beginning with 'U'. Write the updated anchor table (BARCODE, ANCHOR_READ, ANCHOR_SEQ, READ_SUPPORT, CHROM) back to sample.telomap.anchors.tsv. Unambiguous placement depends on subtelomeric sequence length — restriction trimming with RsaI/HinfI can leave subtelomeric regions too short for unambiguous identification against T2T-CHM13.

## Related tools

- **telomap** (Identifies telomeric reads, clusters subtelomeric regions into consensus anchor sequences, and maps those anchors to chromosome ends based on the T2T-CHM13 assembly) — https://github.com/cytham/telomap
- **T2T-CHM13** (Telomere-to-telomere human reference assembly providing complete subtelomeric sequence used as the mapping target for chromosome-end assignment)
- **pbmm2** (Minimap2-based PacBio aligner available for aligning long reads to the reference assembly) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa  # runs the full telomap workflow including anchor-to-T2T-CHM13 chromosome-end mapping, producing sample.telomap.anchors.tsv
```

## Evaluation signals

- Every anchor cluster row in sample.telomap.anchors.tsv carries a CHROM value that is either a T2T-CHM13 chromosome end or an unmappable ID beginning with 'U'
- Output table preserves the schema BARCODE, ANCHOR_READ, ANCHOR_SEQ, READ_SUPPORT, CHROM with one row per anchor cluster
- Mappable chromosome ends (e.g., Chr. 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, 17p) are unique per patient sample and consistent with per-nucleotide profiles aggregated across reads of each end
- READ_SUPPORT counts are unchanged relative to the input anchor table (mapping only adds CHROM assignments, not new clusters)

## Limitations

- Subtelomeric regions from HiFi reads trimmed with RsaI and HinfI can be too short for unambiguous identification of specific chromosome ends against T2T-CHM13, forcing 'U' (unmappable) labels
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within telomeric variant sequences (TVSs), reducing usable subtelomeric anchor sequence
- The method is incompatible with ALT cells whose very long telomeres exceed PacBio HiFi read lengths, so anchors from such telomeres cannot be generated or mapped

## Evidence

- [methods] Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [readme] Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences: "Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences
* Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [discussion] Subtelomeric regions may be too short for unambiguous mapping: "one potential drawback of trimming using RsaI and HinfI is that the resulting subtelomeric region from the HiFi reads could be too short for unambiguous identification of specific chromosome ends"
- [readme] anchors.tsv output file description: "sample.telomap.anchors.tsv | Table containing details of subtelomeric anchor sequences for chromosome-end mapping"
