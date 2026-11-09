---
name: subtelomeric-segment-clustering
description: Use when telomeric reads from telomap must be assigned to specific chromosome
  ends so per-end analyses such as TVS signature plots can be built, by clustering
  their 5' subtelomeric segments into anchor groups.
when_to_use_negative:
- The subtelomeric region extracted from the HiFi read is too short (e.g. after aggressive
  RsaI/HinfI trimming) for unambiguous identification of specific chromosome ends
  against T2T-CHM13.
- Input is raw WGS or telobait reads that have not yet passed telomap's telomere-identification
  step — the clustering step requires telomeric reads, not raw long reads.
- The goal is measuring telomere length of ALT cells, whose very long telomeres exceed
  PacBio HiFi read length and are incompatible with this workflow.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: telomap
  role: Identifies telomeric reads and clusters their subtelomeric regions to generate
    unique consensus anchor sequences for chromosome-end mapping
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: Sequence alignment used for identity-based clustering/mapping of subtelomeric
    anchor sequences
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:dcf17f5282a4b7c11f42b54f8b98ecc6c4dba00685ea2ee619fc654183b76745
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# subtelomeric-segment-clustering

## Summary

Clusters the non-telomeric 5' subtelomeric segments of telomeric long reads (from telomap) to group reads by chromosome end, building a 100 bp per-cluster consensus anchor that supports chromosome-end mapping and per-end TVS signature plots.

## When to use

Use when telomeric reads from telomap must be assigned to specific chromosome ends so per-end analyses such as TVS signature plots can be built, by clustering their 5' subtelomeric segments into anchor groups.

## When NOT to use

- The subtelomeric region extracted from the HiFi read is too short (e.g. after aggressive RsaI/HinfI trimming) for unambiguous identification of specific chromosome ends against T2T-CHM13.
- Input is raw WGS or telobait reads that have not yet passed telomap's telomere-identification step — the clustering step requires telomeric reads, not raw long reads.
- The goal is measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi read length and are incompatible with this workflow.

## Inputs

- Telomeric long reads (FASTA/FASTQ/BAM) identified by the upstream telomere-identification step of telomap
- 5' non-telomeric subtelomeric segments extracted from each telomeric read
- T2T-CHM13 human genome assembly (for subsequent chromosome-end mapping of anchors)

## Outputs

- sample.telomap.anchors.tsv with per-cluster columns including BARCODE, representative anchor READ_ID, 100 bp consensus ANCHOR_SEQ, READ_SUPPORT count, and CHROM (mapped chromosome end or 'U'-prefixed unmappable cluster ID)
- Grouped telomeric reads per cluster, used to build TVS signature plots for specific telomere ends

## How to apply

Load the telomeric reads identified by the upstream telomere-identification step of telomap and extract the 5' non-telomeric subtelomeric segment from each read. Cluster the subtelomeric segments using a minimum sequence identity threshold of 95% and a minimum alignment length of 100 bp, which groups reads likely originating from the same chromosomal end. For each cluster, build a 100 bp consensus anchor sequence by taking the most common base at each position, and record per-cluster details (barcode, representative anchor read ID, consensus anchor sequence, number of supporting reads) in the anchors table. Downstream, map the subtelomeric anchor sequences to chromosome ends based on the T2T-CHM13 assembly; clusters that cannot be unambiguously mapped receive an unmappable cluster ID beginning with 'U'. These cluster groupings provide the balanced, haplotype-like read sets (e.g. via K-means classification) used to construct TVS signature plots for specific telomere ends of each patient sample.

## Related tools

- **telomap** (Identifies telomeric reads and clusters their subtelomeric regions to generate unique consensus anchor sequences for chromosome-end mapping) — https://github.com/cytham/telomap
- **pbmm2** (Sequence alignment used for identity-based clustering/mapping of subtelomeric anchor sequences) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap wgs /path/to/reads.fastq.gz /path/to/work_dir -t 24
```

## Evaluation signals

- sample.telomap.anchors.tsv exists and every row carries a consensus ANCHOR_SEQ.
- Every anchors-table row has a READ_SUPPORT count (number of supporting reads) greater than zero.
- Every row's CHROM value is either a mapped chromosomal end or an unmappable cluster ID beginning with 'U'.
- Each consensus ANCHOR_SEQ is 100 bp, built by majority base at each position from the cluster's subtelomeric segments.
- Clustered reads meet the 95% minimum sequence identity and 100 bp minimum alignment length thresholds.

## Limitations

- Trimming genomic DNA with RsaI and HinfI can yield subtelomeric regions too short for unambiguous identification of specific chromosome ends against T2T-CHM13.
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length, which affects the reads available for anchoring.
- Reducing the --barscore minimum alignment score fraction may lead to multiple barcode mapping per read, causing high read omission in telobait mode.

## Evidence

- [methods] Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences: "Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences"
- [methods] Map anchors to T2T-CHM13: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [readme] Telomap identifies and analyses telomeric reads: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [readme] Anchors output table description: "sample.telomap.anchors.tsv | Table containing details of subtelomeric anchor sequences for chromosome-end mapping"
- [discussion] Subtelomeric regions may be too short for unambiguous end identification: "one potential drawback of trimming using RsaI and HinfI is that the resulting subtelomeric region from the HiFi reads could be too short for unambiguous identification of specific chromosome ends"
