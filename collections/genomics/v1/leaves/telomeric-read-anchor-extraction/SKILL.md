---
name: telomeric-read-anchor-extraction
description: Use when you have telomeric reads (FASTQ/BAM from WGS or telobait-captured
  PacBio HiFi data) already flagged by telomap and need to group them by chromosome
  end via subtelomeric consensus anchors before per-end analyses.
when_to_use_negative:
- Input reads have not yet been screened for telomeric sequence by telomap (e.g. raw
  whole-genome long reads without the telomere-identification step)
- Subtelomeric regions are too short for unambiguous chromosome-end identification
  (e.g. after RsaI/HinfI trimming) against T2T-CHM13, so anchors would map ambiguously
- Analysis goal is ALT-cell telomere length, whose very long telomeres exceed PacBio
  HiFi read length and are incompatible with this pipeline
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: telomap
  role: Runs the clustering of the subtelomeric region of telomeric reads, generates
    unique consensus anchor sequences, and writes sample.telomap.anchors.tsv
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: PacBio-aware minimap2 wrapper used for aligning PacBio HiFi reads/anchor sequences
    against the reference during chromosome-end mapping
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:73995dd331040d4751fd6c1a7b21aa436e428d6f29b44717bbe349d4e8e3fa6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# telomeric-read-anchor-extraction

## Summary

Clusters the 5' non-telomeric subtelomeric segments of telomeric reads identified by telomap and builds a 100 bp consensus anchor per cluster, enabling individual telomeric reads to be assigned to specific chromosome ends for per-end analyses such as TVS signature plots.

## When to use

Use when you have telomeric reads (FASTQ/BAM from WGS or telobait-captured PacBio HiFi data) already flagged by telomap and need to group them by chromosome end via subtelomeric consensus anchors before per-end analyses.

## When NOT to use

- Input reads have not yet been screened for telomeric sequence by telomap (e.g. raw whole-genome long reads without the telomere-identification step)
- Subtelomeric regions are too short for unambiguous chromosome-end identification (e.g. after RsaI/HinfI trimming) against T2T-CHM13, so anchors would map ambiguously
- Analysis goal is ALT-cell telomere length, whose very long telomeres exceed PacBio HiFi read length and are incompatible with this pipeline

## Inputs

- Telomeric long reads (FASTQ/FASTA/BAM/pacbio-bam) identified by the upstream telomap telomere-identification step
- T2T-CHM13 human genome assembly for mapping consensus anchors to chromosome ends
- telomap work directory (e.g. /path/to/work_dir) with sample name prefix

## Outputs

- sample.telomap.anchors.tsv with per-cluster ANCHOR_SEQ, READ_SUPPORT, CHROM, barcode, and representative read ID
- Cluster assignments of telomeric reads usable as grouped inputs for TVS signature plots

## How to apply

After the upstream telomap step identifies reads containing telomeric sequences (TELOMERE_START defined by at least two consecutive 5'-TTAGGGTTAGGG-3' repeats), extract the 5' non-telomeric subtelomeric segment from each telomeric read. Cluster these subtelomeric segments with a minimum sequence identity threshold of 95% and a minimum alignment length of 100 bp. For each cluster, build a 100 bp consensus anchor sequence by taking the most common base at each position, then map anchor sequences to chromosome ends using the T2T-CHM13 human genome assembly. Record per-cluster details (barcode, representative anchor read ID, consensus anchor sequence, number of supporting reads) in the anchors table; clusters that cannot be unambiguously assigned receive an unmappable cluster ID beginning with 'U'. Downstream, reads grouped into clusters (with K-means-based haplotype classification and balanced numbers) supply the per-chromosome-end groups used for TVS signature plots.

## Related tools

- **telomap** (Runs the clustering of the subtelomeric region of telomeric reads, generates unique consensus anchor sequences, and writes sample.telomap.anchors.tsv) — https://github.com/cytham/telomap
- **pbmm2** (PacBio-aware minimap2 wrapper used for aligning PacBio HiFi reads/anchor sequences against the reference during chromosome-end mapping) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap wgs /path/to/reads.fastq.gz /path/to/work_dir -t 24 -n sample
```

## Evaluation signals

- sample.telomap.anchors.tsv exists and every row carries a consensus ANCHOR_SEQ, a READ_SUPPORT count, and a CHROM value that is either a mapped chromosomal end or an unmappable cluster ID beginning with 'U'
- Each anchor sequence is 100 bp long, built from the most common base at each position of its cluster
- Clustered segments satisfy the thresholds of ≥95% sequence identity and ≥100 bp minimum alignment length
- TELOMERE_START positions in the main table correspond to reads with at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')
- Anchor sequences map to expected chromosome ends under the T2T-CHM13 assembly

## Limitations

- Trimming genomic DNA with RsaI and HinfI can leave subtelomeric regions too short for unambiguous identification of specific chromosome ends against T2T-CHM13, producing unmappable ('U') clusters
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Reducing the --barscore threshold may lead to multiple barcode mappings per read, causing high read omission upstream of anchor building

## Evidence

- [methods] workflow step rationale: "Cluster subtelomeric region of telomeric reads to generate unique consensus anchor sequences"
- [methods] mapping reference: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
- [methods] telomere start definition: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] anchors output file: "sample.telomap.anchors.tsv | Table containing details of subtelomeric anchor sequences for chromosome-end mapping"
- [full_text] TVS grouping for per-end plots: [evidence span withheld — anchor status: not_found]
