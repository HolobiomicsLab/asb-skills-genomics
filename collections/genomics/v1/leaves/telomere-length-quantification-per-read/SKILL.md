---
name: telomere-length-quantification-per-read
description: Use when PacBio HiFi long reads (telobait-captured or WGS) must be converted
  into per-read telomere length distributions, e.g. comparing short-telomere enrichment
  across cell lines such as T24 vs HCT116, 293T, IMR90, WI38.
when_to_use_negative:
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio
  HiFi DNA polymerase processivity
- Unambiguous chromosome-end assignment when subtelomeric regions are too short after
  restriction trimming (e.g. RsaI/HinfI) to map against T2T-CHM13
- Vanilla long-read data without telobait capture is present but the workflow requires
  wet-lab-captured data — use wgs mode instead of misapplying telobait mode
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: telomap
  role: 'Primary tool: identifies telomeric reads, demarcates TELOMERE_START/TELOMERE_END,
    computes RAW_TELOMERE_LEN and TELOMERE_LEN, generates anchor mapping and analytic
    plots'
  repo: https://github.com/cytham/telomap
- name: PacBio high fidelity (HiFi) sequencing platform
  role: 'Data source: single-molecule real-time sequencing providing long HiFi reads
    that enable nucleotide-resolution telomere length measurement'
- name: pbmm2
  role: PacBio-native minimap2 wrapper for aligning PacBio long reads referenced in
    the article's tooling
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:b5fce5cd072aea4bfa5a116dd95480b9cba42890f4f11548cec73a4a946965ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - PacBio high fidelity (HiFi) sequencing platform
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# telomere-length-quantification-per-read

## Summary

Per-read quantification of telomere lengths at nucleotide resolution from PacBio HiFi telobait-captured or WGS long-read data using telomap, which identifies telomeric reads, demarcates the telomeric region, and computes both raw and canonical-repeat-only lengths.

## When to use

Use when PacBio HiFi long reads (telobait-captured or WGS) must be converted into per-read telomere length distributions, e.g. comparing short-telomere enrichment across cell lines such as T24 vs HCT116, 293T, IMR90, WI38.

## When NOT to use

- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity
- Unambiguous chromosome-end assignment when subtelomeric regions are too short after restriction trimming (e.g. RsaI/HinfI) to map against T2T-CHM13
- Vanilla long-read data without telobait capture is present but the workflow requires wet-lab-captured data — use wgs mode instead of misapplying telobait mode

## Inputs

- Demultiplexed telobait-captured PacBio HiFi reads (fasta/fastq/bam/pacbio-bam) per sample
- capture_oligo.fa — capture oligo sequence (sequence after the barcode on the telobait)
- barcodes.fa — sample barcode sequences embedded within the telobaits
- Vanilla WGS long-read data (for wgs mode alternative)

## Outputs

- sample.telomap.tsv — per-read table with 13 columns (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_END_MOTIF, TRF_COUNT, CHROM, TELOMERE_START, TELOMERE_END)
- sample.telomap.anchors.tsv — subtelomeric anchor sequences for chromosome-end mapping
- plots directory — analytic plots of per-sample telomere length distribution
- plots_QC directory — QC plots

## How to apply

Run telomap in telobait mode on demultiplexed telobait-captured HiFi reads, passing the capture_oligo.fa (-c) and barcodes.fa (-b) files bundled with the package and specifying threads (-t 24). Telomap detects telomeric reads using the default TTAGGG motif (-m TTAGGG) and demarcates each read's telomeric region via TELOMERE_START, the 1-based 5' position containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3'), and TELOMERE_END, the 1-based 3' position adjacent to the telobait sequence. RAW_TELOMERE_LEN is TELOMERE_END - TELOMERE_START + 1, while TELOMERE_LEN is motif count times the 6 bp motif length, excluding all non-telomeric variant sequences (TVSs) within the raw region. Telomap also clusters the subtelomeric region of telomeric reads into consensus anchor sequences and maps them to chromosome ends based on the T2T-CHM13 assembly, then emits QC and analytic plots for the per-sample telomere length distribution.

## Related tools

- **telomap** (Primary tool: identifies telomeric reads, demarcates TELOMERE_START/TELOMERE_END, computes RAW_TELOMERE_LEN and TELOMERE_LEN, generates anchor mapping and analytic plots) — https://github.com/cytham/telomap
- **PacBio high fidelity (HiFi) sequencing platform** (Data source: single-molecule real-time sequencing providing long HiFi reads that enable nucleotide-resolution telomere length measurement)
- **pbmm2** (PacBio-native minimap2 wrapper for aligning PacBio long reads referenced in the article's tooling) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- sample.telomap.tsv exists with all 13 declared columns populated per read
- TELOMERE_START corresponds to the 1-based position containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')
- TELOMERE_LEN equals motif count × 6 bp and excludes all non-telomeric sequences within RAW_TELOMERE_LEN
- Distribution plots reproduce the expected finding, e.g. over-representation of extremely short (<1000 bp) telomeres in T24 cells relative to HCT116, 293T, IMR90 (PD47), and WI38 (PD40)
- Anchors from sample.telomap.anchors.tsv map to chromosome ends based on the T2T-CHM13 assembly

## Limitations

- Incompatible with measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Trimming with RsaI and HinfI may yield subtelomeric regions too short for unambiguous identification of specific chromosome ends against T2T-CHM13
- Reducing --barscore may lead to multiple barcode mapping per read, causing high read omission

## Evidence

- [methods] Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data.: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [methods] TELOMERE_START definition: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] TELOMERE_LEN column definition: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [discussion] T24 extremely short telomeres finding: "our method has identified the over-representation of extremely short telomeres in T24 cells"
