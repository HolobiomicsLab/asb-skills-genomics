---
name: telomere-variant-sequence-exclusion-from-length
description: Use when per-read telomere lengths from PacBio HiFi telomeric reads must
  exclude non-canonical variant repeats so that length reflects continuous canonical
  (TTAGGG)n tracts rather than raw subtelomere-to-bait span.
when_to_use_negative:
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio
  HiFi DNA polymerase processivity
- Vanilla whole-genome sequencing long-read data without telobait capture (use telomap
  WGS mode instead; OLIGO/BARCODE are NA)
- When raw telomeric-region span including TVSs is the desired quantity (then use
  RAW_TELOMERE_LEN directly)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: telomap
  role: Identifies telomeric reads, demarcates TELOMERE_START/TELOMERE_END, counts
    canonical TTAGGG motifs, and reports RAW_TELOMERE_LEN and TVS-excluded TELOMERE_LEN
    in sample.telomap.tsv
  repo: https://github.com/cytham/telomap
- name: PacificBiosciences/pbmm2
  role: Minimap2 SMRT wrapper used for alignment of PacBio HiFi reads underlying subtelomeric
    anchor mapping
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:c578495cbfd2c79b4f891556369da7dc977aefad2ef380a52148ddac0ef16d6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - PacificBiosciences/pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# telomere-variant-sequence-exclusion-from-length

## Summary

Compute telomere length at nucleotide resolution from PacBio HiFi telobait-captured reads by counting canonical 5'-TTAGGG-3' repeats and excluding interspersed telomeric variant sequences (TVSs) from the raw demarcated telomeric region, yielding both RAW_TELOMERE_LEN and the corrected TELOMERE_LEN per read.

## When to use

Use when per-read telomere lengths from PacBio HiFi telomeric reads must exclude non-canonical variant repeats so that length reflects continuous canonical (TTAGGG)n tracts rather than raw subtelomere-to-bait span.

## When NOT to use

- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity
- Vanilla whole-genome sequencing long-read data without telobait capture (use telomap WGS mode instead; OLIGO/BARCODE are NA)
- When raw telomeric-region span including TVSs is the desired quantity (then use RAW_TELOMERE_LEN directly)

## Inputs

- Demultiplexed telobait-captured PacBio HiFi reads (FASTQ/FASTA/BAM)
- capture_oligo.fa (sequence after the barcode on the telobait)
- barcodes.fa (sample barcode sequences embedded in telobaits)
- telomeric motif definition (default TTAGGG, 6 bp)

## Outputs

- sample.telomap.tsv with 13 columns per read (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_END_MOTIF, TRF_COUNT, CHROM, TELOMERE_START, TELOMERE_END)
- plots directory with per-sample telomere length distributions at nucleotide resolution
- sample.telomap.anchors.tsv with subtelomeric anchor sequences for chromosome-end mapping

## How to apply

After telomap (telobait mode) identifies telomeric reads using the default TTAGGG motif, demarcate the telomeric region per read: TELOMERE_START is the 1-based 5' read position containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3') and TELOMERE_END is the 1-based 3' position adjacent to the telobait sequence. Record RAW_TELOMERE_LEN as TELOMERE_END - TELOMERE_START + 1. Then compute TELOMERE_LEN as the telomeric motif count multiplied by the motif length (6 bp), which excludes all non-telomeric TVS sequences embedded within the raw region. This distinction matters because TVSs disrupt the continuity of canonical repeats and affect shelterin binding, so only the canonical-repeat length is reported. Per-read results are exported to sample.telomap.tsv with both length columns plus TELOMERE_END_MOTIF and TRF_COUNT for auditing.

## Related tools

- **telomap** (Identifies telomeric reads, demarcates TELOMERE_START/TELOMERE_END, counts canonical TTAGGG motifs, and reports RAW_TELOMERE_LEN and TVS-excluded TELOMERE_LEN in sample.telomap.tsv) — https://github.com/cytham/telomap
- **PacificBiosciences/pbmm2** (Minimap2 SMRT wrapper used for alignment of PacBio HiFi reads underlying subtelomeric anchor mapping) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa -m TTAGGG
```

## Evaluation signals

- sample.telomap.tsv exists with all 13 declared columns populated per read
- For every read, TELOMERE_LEN <= RAW_TELOMERE_LEN, with equality only when no TVSs interrupt the region
- TELOMERE_LEN is a multiple of 6 bp (motif count × motif length) for the default TTAGGG motif
- TELOMERE_START position contains at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')
- Length-distribution plots reproduce expected biology, e.g. over-representation of extremely short (<1000 bp) telomeres in T24 cells

## Limitations

- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length; careful selection of trimming enzymes is necessary
- Trimming with RsaI and HinfI can leave subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13
- Method is incompatible with ALT cells whose very long telomeres exceed PacBio HiFi read lengths
- Reducing --barscore below 1 may lead to multiple barcode mapping per read and high read omission

## Evidence

- [methods] TELOMERE_START definition requiring at least two consecutive telomeric repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] TELOMERE_LEN excludes TVSs: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [abstract] TVSs disrupt canonical repeats and shelterin binding: "The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes"
- [discussion] T24 extremely short telomeres reproduced: "our method has identified the over-representation of extremely short telomeres in T24 cells"
- [readme] telomap purpose: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
