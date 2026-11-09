---
name: long-read-sequence-identity-clustering
description: Use when long reads identified as telomeric (PacBio HiFi WGS or telobait-captured
  FASTQ/BAM) must be grouped by their subtelomeric region so per-end analyses such
  as TVS signature plots can be built for specific telomeres.
when_to_use_negative:
- The input reads have not yet been filtered for telomeric content — run telomap's
  telomere-identification step first
- Restriction trimming (e.g. RsaI/HinfI) left subtelomeric regions too short for unambiguous
  chromosome-end identification against T2T-CHM13
- Telomeres are extremely long (ALT cells), where PacBio HiFi polymerase processivity
  limits read length and this workflow is incompatible
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: telomap
  role: Identifies telomeric long reads, clusters their subtelomeric regions into
    candidate chromosome-end groups, and emits the consensus anchor table
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: SMRT wrapper for minimap2 used in the PacBio alignment environment supporting
    the long-read data analyzed by telomap
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:36fe2c7f655df8f1c65b48f6aa5d566187b0b17f5ef0f838f25e0b102a4b2c53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# long-read-sequence-identity-clustering

## Summary

Clusters the 5' non-telomeric subtelomeric segments of telomeric long reads (via telomap) by sequence identity to group reads originating from the same chromosome end, then builds a per-cluster consensus anchor sequence for downstream assignment to specific chromosomal ends against T2T-CHM13.

## When to use

Use when long reads identified as telomeric (PacBio HiFi WGS or telobait-captured FASTQ/BAM) must be grouped by their subtelomeric region so per-end analyses such as TVS signature plots can be built for specific telomeres.

## When NOT to use

- The input reads have not yet been filtered for telomeric content — run telomap's telomere-identification step first
- Restriction trimming (e.g. RsaI/HinfI) left subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13
- Telomeres are extremely long (ALT cells), where PacBio HiFi polymerase processivity limits read length and this workflow is incompatible

## Inputs

- Telomeric long reads (FASTA/FASTQ/BAM/PacBio BAM) produced by the upstream telomere-identification step of telomap
- 5' non-telomeric subtelomeric segments extracted per telomeric read
- T2T-CHM13 human genome assembly for anchor-to-chromosome-end mapping

## Outputs

- sample.telomap.anchors.tsv with per-cluster BARCODE, representative READ ID, 100 bp ANCHOR_SEQ, READ_SUPPORT, and CHROM (mapped chromosomal end or 'U'-prefixed unmappable cluster ID)
- Clustered read assignments used to construct per-end TVS signature plots

## How to apply

Run telomap (WGS or telobait mode) so that telomeric reads are first identified by the presence of consecutive telomeric repeats (e.g. 5'-TTAGGGTTAGGG-3' defining TELOMERE_START). Extract the 5' non-telomeric subtelomeric segment from each telomeric read and cluster these segments using a minimum sequence identity threshold of 95% and a minimum alignment length of 100 bp. For each cluster, construct a 100 bp consensus anchor sequence by taking the most common base at each position, and record per-cluster details (barcode, representative anchor read ID, consensus ANCHOR_SEQ, READ_SUPPORT count, CHROM) in the anchors table. Map each consensus anchor to chromosome ends using the T2T-CHM13 assembly; clusters that cannot be unambiguously assigned receive an unmappable cluster ID beginning with 'U'. Reads classified into these haplotype-like clusters are then balanced in numbers to build TVS signature plots for specific telomere ends of each patient sample.

## Related tools

- **telomap** (Identifies telomeric long reads, clusters their subtelomeric regions into candidate chromosome-end groups, and emits the consensus anchor table) — https://github.com/cytham/telomap
- **pbmm2** (SMRT wrapper for minimap2 used in the PacBio alignment environment supporting the long-read data analyzed by telomap) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap wgs /path/to/reads.fastq.gz /path/to/work_dir -t 24
```

## Evaluation signals

- sample.telomap.anchors.tsv exists and every row carries a consensus ANCHOR_SEQ
- Every anchors-table row has a READ_SUPPORT count > 0
- Every CHROM value is either a mapped chromosomal end or an unmappable cluster ID beginning with 'U'
- Consensus anchors are 100 bp, built by majority base at each position
- Clustered reads for a given chromosome end support the expected number of telomere ends when balanced for TVS signature plotting

## Limitations

- Subtelomeric regions trimmed with RsaI and HinfI may be too short for unambiguous identification of specific chromosome ends against T2T-CHM13
- Clusters that cannot be unambiguously mapped remain 'U'-prefixed IDs rather than chromosomal ends
- Incompatible with ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase processivity

## Evidence

- [methods] Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences: "Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences"
- [readme] sample.telomap.anchors.tsv output description: "sample.telomap.anchors.tsv | Table containing details of subtelomeric anchor sequences for chromosome-end mapping"
- [discussion] Subtelomere too short caveat: "the resulting subtelomeric region from the HiFi reads could be too short for unambiguous identification of specific chromosome ends"
- [methods] Anchor mapping reference assembly: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
