---
name: single-cell-read-deduplication
description: Use when demultiplexing HIT-scISOseq or 10x Genomics-based long-read
  scRNA-seq FLNC reads requires correcting sequencing-error-derived cellBCs and merging
  UMI duplicates per gene/cell.
when_to_use_negative:
- Input reads carry no cell barcodes or UMIs (e.g. bulk Iso-Seq or TSO-artifact reads
  lacking barcode structure)
- A short-read 10x scRNA-seq dataset is available that CellRanger can process natively;
  cellBC_UMI_corrector instead wraps CellRanger logic for long-read data without short-read
  guidance
- Barcode structure follows a non-10x design not represented in the 3M-february-2018
  whitelist
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Hosts the cellBC_UMI_corrector module performing whitelist-based barcode correction
    and UMI collapse on FLNC reads
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Source of the wrapped cell barcode and UMI correction function reused inside
    cellBC_UMI_corrector
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:c61939dc655e518d772c9b01d2da7f01e67be219cd3d9c007a6904efd3002647
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-read-deduplication

## Summary

Correct single-cell barcodes against a whitelist and collapse UMI duplicates from long-read single-cell RNA-seq data using scISA-Tools' cellBC_UMI_corrector, which wraps CellRanger's cellBC/UMI correction logic for HIT-scISOseq FLNC reads.

## When to use

Use when demultiplexing HIT-scISOseq or 10x Genomics-based long-read scRNA-seq FLNC reads requires correcting sequencing-error-derived cellBCs and merging UMI duplicates per gene/cell.

## When NOT to use

- Input reads carry no cell barcodes or UMIs (e.g. bulk Iso-Seq or TSO-artifact reads lacking barcode structure)
- A short-read 10x scRNA-seq dataset is available that CellRanger can process natively; cellBC_UMI_corrector instead wraps CellRanger logic for long-read data without short-read guidance
- Barcode structure follows a non-10x design not represented in the 3M-february-2018 whitelist

## Inputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads with cellBC/UMI and base qualities)
- 3M-february-2018.txt (10x Genomics 737k whitelist barcode list)
- flnc.filtered.tmap (gffcompare-filtered isoform assignment table per FLNC read)

## Outputs

- flnc.cellBC_UMI_correction.xls (corrected cellBC/UMI assignments joined to isoform assignments per FLNC read)

## How to apply

Load the FLNC barcode/UMI fastq (isoseq_flnc.BarcodeUMI.fastq), the 10x Genomics whitelist (3M-february-2018.txt), and the filtered isoform assignment table (flnc.filtered.tmap). First apply UMI quality filtering: discard UMIs that are homopolymers (e.g. AAAAAAAAAA), contain N bases, or contain any base with base quality <10. Correct each observed cell barcode by matching it to the nearest whitelist barcode within 1 Hamming distance, replacing it with the highest-posterior whitelist barcode only when the posterior probability exceeds 0.975; otherwise drop the read. Then collapse UMIs per gene/cell by merging UMIs within 1 Hamming distance to remove duplicates arising from sequencing errors. Finally join the corrected cellBC/UMI assignments with the tmap isoform assignments per FLNC read and write flnc.cellBC_UMI_correction.xls. The rationale is that long PacBio HiFi reads still carry barcode/UMI errors that would inflate cell and molecule counts, so CellRanger-style probabilistic correction is applied independently of short-read data.

## Related tools

- **scISA-Tools** (Hosts the cellBC_UMI_corrector module performing whitelist-based barcode correction and UMI collapse on FLNC reads) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Source of the wrapped cell barcode and UMI correction function reused inside cellBC_UMI_corrector)

## Examples

```
/path/cellranger-3.1.0/miniconda-cr-cs/4.3.21-miniconda-cr-cs-c10/bin/python cellBC_UMI_corrector.py --input isoseq_flnc.BarcodeUMI.fastq --whitelist 3M-february-2018.txt --tmap flnc.filtered.tmap
```

## Evaluation signals

- flnc.cellBC_UMI_correction.xls exists and every emitted cell barcode is a member of the 3M-february-2018 whitelist (or the read was dropped)
- The number of corrected records is consistent with the number of FLNC entries in isoseq_flnc.BarcodeUMI.fastq passing UMI quality filters (non-homopolymer, no N, all base qualities ≥10)
- All corrected cellBCs are within 1 Hamming distance of their original observed barcodes, and each replacement had posterior probability >0.975
- No two UMIs within 1 Hamming distance remain unmerged for the same gene/cell combination after collapse
- Downstream isoform count matrix reflects deduplication: scIsoform_matrix/scGene_matrix counts per cell match the collapsed UMI-per-gene totals

## Limitations

- Correction is restricted to whitelisted 10x Genomics barcodes within 1 Hamming distance; barcodes further than 1 edit from any whitelist member are dropped, discarding reads
- The UMI quality filter rejects any UMI containing a base with base quality <10, a homopolymer, or an N, which can bias against low-quality molecules
- Hamming-distance-based UMI collapse may merge genuinely distinct UMIs that differ by a single true error-free base (collision risk in high-UMI-diversity cells)

## Evidence

- [discussion] UMI quality filtering thresholds: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] cellBC correction posterior threshold: [evidence span withheld — anchor status: not_found]
- [methods] cellBC_UMI_corrector wraps CellRanger: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [readme] no short-read guidance needed: "these methods using long reads data independently, no need additional short read data for guiding"
- [methods] whitelist file location: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
