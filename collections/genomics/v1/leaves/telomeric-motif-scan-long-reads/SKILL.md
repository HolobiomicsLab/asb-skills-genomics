---
name: telomeric-motif-scan-long-reads
description: Use when you have PacBio HiFi long reads from telobait-captured or whole-genome
  sequencing data and need to identify reads containing telomeric (TTAGGG) repeats,
  measure telomere length at single-nucleotide resolution, and characterize telomeric
  variant sequences (TVS) distribution.
when_to_use_negative:
- Measuring telomere length of ALT (alternative lengthening of telomeres) cells whose
  very long telomeres exceed PacBio HiFi DNA polymerase processivity limits
- Input data is short-read sequencing (e.g., Illumina) — the workflow is designed
  specifically for long-read platforms
- Subtelomeric regions are expected to be too short (e.g., after aggressive restriction
  enzyme trimming with RsaI and HinfI) for unambiguous chromosome-end identification
  against T2T-CHM13
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: telomap
  role: Primary tool to identify telomeric reads, compute telomere length at nucleotide
    resolution, demarcate telomeric region boundaries, detect TVS motifs, and generate
    QC/analytic plots from WGS or telobait-captured long-read data
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: SMRT sequencing aligner (minimap2 wrapper for PacBio data) used for aligning
    long reads during the telomere analysis workflow
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:f9754e21d2d1edef414b26a3e64590d641dee292df414640fafa7f8bfb652fce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# telomeric-motif-scan-long-reads

## Summary

Scans long-read (PacBio HiFi) sequencing data for telomeric sequences using the telomap tool, identifying telomeric reads and measuring telomere lengths at nucleotide resolution by demarcating telomeric regions via consecutive canonical TTAGGG repeats and quantifying both raw and motif-count-based lengths per read.

## When to use

Use when you have PacBio HiFi long reads from telobait-captured or whole-genome sequencing data and need to identify reads containing telomeric (TTAGGG) repeats, measure telomere length at single-nucleotide resolution, and characterize telomeric variant sequences (TVS) distribution.

## When NOT to use

- Measuring telomere length of ALT (alternative lengthening of telomeres) cells whose very long telomeres exceed PacBio HiFi DNA polymerase processivity limits
- Input data is short-read sequencing (e.g., Illumina) — the workflow is designed specifically for long-read platforms
- Subtelomeric regions are expected to be too short (e.g., after aggressive restriction enzyme trimming with RsaI and HinfI) for unambiguous chromosome-end identification against T2T-CHM13

## Inputs

- PacBio HiFi long-read FASTQ/FASTA/BAM files (telobait-captured or whole-genome sequencing data)
- capture_oligo.fa (capture oligo sequence FASTA, required for telobait mode)
- barcodes.fa (sample barcode FASTA, required for telobait mode)
- T2T-CHM13 human genome assembly (for chromosome-end mapping)

## Outputs

- sample.telomap.tsv (main per-read table with columns: RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_END_MOTIF, TRF_COUNT, CHROM, TELOMERE_START, TELOMERE_END)
- sample.telomap.anchors.tsv (subtelomeric anchor sequences for chromosome-end mapping)
- plots directory (analytic plots including telomere length distribution per sample)
- plots_QC directory (quality control plots)

## How to apply

Run telomap in the appropriate mode: 'telobait' mode for telobait-captured reads (requires capture_oligo.fa and barcodes.fa) or 'wgs' mode for vanilla whole-genome long reads. Launch with 'telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c capture_oligo.fa -b barcodes.fa'. Telomap identifies telomeric reads using the default TTAGGG motif, demarcates TELOMERE_START as the 1-based 5' position containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3') and TELOMERE_END as the 1-based 3' position adjacent to the telobait sequence. RAW_TELOMERE_LEN is computed as TELOMERE_END - TELOMERE_START + 1, and TELOMERE_LEN is calculated as motif count × 6 bp after excluding all non-telomeric variant sequences within the raw region. Cluster subtelomeric regions of telomeric reads to generate consensus anchor sequences, then map these anchors to chromosome ends using the T2T-CHM13 human genome assembly.

## Related tools

- **telomap** (Primary tool to identify telomeric reads, compute telomere length at nucleotide resolution, demarcate telomeric region boundaries, detect TVS motifs, and generate QC/analytic plots from WGS or telobait-captured long-read data) — https://github.com/cytham/telomap
- **pbmm2** (SMRT sequencing aligner (minimap2 wrapper for PacBio data) used for aligning long reads during the telomere analysis workflow) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- sample.telomap.tsv exists with all 13 declared columns (RNAME through TELOMERE_END) populated per read
- TELOMERE_START is defined as the 1-based position containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')
- RAW_TELOMERE_LEN equals TELOMERE_END - TELOMERE_START + 1
- TELOMERE_LEN equals telomeric motif count × 6 bp, with all non-telomeric variant sequences excluded from the raw region
- Distribution plots reproduce expected telomere length profiles (e.g., over-representation of extremely short <1000 bp telomeres in T24 cells relative to other cell lines)

## Limitations

- Trimming telomere-containing genomic DNA is necessary because PacBio HiFi maximum read length is limited by DNA polymerase processivity
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Trimming with RsaI and HinfI may yield subtelomeric regions too short for unambiguous identification of specific chromosome ends against T2T-CHM13
- The method is incompatible with measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity
- Careful selection of restriction enzymes for genomic DNA trimming is necessary

## Evidence

- [methods] Telomap identifies telomeric reads using the default TTAGGG motif: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [methods] TELOMERE_START requires at least two consecutive telomeric repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] RAW_TELOMERE_LEN vs TELOMERE_LEN distinction: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [readme] Telobait mode invocation from README: "telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa"
- [methods] T2T-CHM13 reference for anchor mapping: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
