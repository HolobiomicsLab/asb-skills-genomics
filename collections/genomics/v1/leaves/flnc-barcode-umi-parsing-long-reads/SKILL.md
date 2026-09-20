---
name: flnc-barcode-umi-parsing-long-reads
description: Use when processing PacBio CCS/FLNC single-cell reads (e.g. isoseq_flnc.BarcodeUMI.fastq
  from classify_by_primer) that contain 10x cellBC and UMI sequences needing whitelist-based
  correction before isoform/gene matrix generation.
when_to_use_negative:
- Input is short-read 10x data already processed by the standard CellRanger mkfastq/count
  pipeline rather than PacBio FLNC reads
- Reads have not yet been classified as full-length by classify_by_primer (no isoseq_flnc.BarcodeUMI.fastq
  with cellBC/UMI present)
- The library contains no cell barcode/UMI structure (e.g. bulk Iso-Seq data), so
  whitelist-based demultiplexing is meaningless
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scISA-Tools (cellBC_UMI_corrector)
  role: Pipeline module that wraps CellRanger's cellBC/UMI correction logic for long
    FLNC reads; performs UMI quality filtering, whitelist barcode correction, and
    writes flnc.cellBC_UMI_correction.xls
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger 3.1.0
  role: Source of the 10x barcode whitelist (3M-february-2018.txt) and the barcode/UMI
    correction strategy reimplemented in cellBC_UMI_corrector
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- name: classify_by_primer
  role: Upstream Perl utility that extracts cell barcodes and UMIs and produces isoseq_flnc.BarcodeUMI.fastq
    from primer-classified CCS reads
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:ef3d4590d7fa9b928c318fdfb241616198f86cb18627154da38c84c23e5d4de4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - CellRanger 3.1.0
  - classify_by_primer
  - scISA-Tools
  - scISA-Tools (cellBC_UMI_corrector)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# flnc-barcode-umi-parsing-long-reads

## Summary

Parse and correct 10x Genomics cell barcodes and UMIs from PacBio full-length non-concatemer (FLNC) reads in the HIT-scISOseq/scISA-Tools pipeline, using a CellRanger-style whitelist matching strategy to assign long reads to individual cells and molecules.

## When to use

Use when processing PacBio CCS/FLNC single-cell reads (e.g. isoseq_flnc.BarcodeUMI.fastq from classify_by_primer) that contain 10x cellBC and UMI sequences needing whitelist-based correction before isoform/gene matrix generation.

## When NOT to use

- Input is short-read 10x data already processed by the standard CellRanger mkfastq/count pipeline rather than PacBio FLNC reads
- Reads have not yet been classified as full-length by classify_by_primer (no isoseq_flnc.BarcodeUMI.fastq with cellBC/UMI present)
- The library contains no cell barcode/UMI structure (e.g. bulk Iso-Seq data), so whitelist-based demultiplexing is meaningless

## Inputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads with cellBC/UMI, from classify_by_primer)
- flnc.filtered.tmap (gffcompare isoform assignment table filtered by filter.tmap.pl)
- 10x Genomics whitelist barcode list (3M-february-2018.txt, from CellRanger 3.1.0 or repo example_data)

## Outputs

- flnc.cellBC_UMI_correction.xls (corrected cell barcode and UMI assignments per FLNC read)
- Whitelist-validated cellBC/UMI annotations joined to isoform assignments for scGene_matrix and scIsoform_matrix

## How to apply

After classify_by_primer generates isoseq_flnc.BarcodeUMI.fastq and gffcompare's tmap is filtered to flnc.filtered.tmap, run the scISA-Tools cellBC_UMI_corrector module, which wraps CellRanger's cell barcode and UMI correction for long reads (no short-read guidance data needed). Apply UMI quality filters first: discard UMIs that are homopolymers (e.g. AAAAAAAAAA), contain N bases, or contain bases with base quality <10. Correct each observed cell barcode by matching it to the nearest barcode in the 10x whitelist (3M-february-2018.txt) within 1 Hamming distance, replacing it with the highest-posterior whitelist barcode when the posterior probability exceeds 0.975; otherwise drop the read. Then join corrected cellBC/UMI assignments with the per-FLNC-read isoform assignments from flnc.filtered.tmap and write flnc.cellBC_UMI_correction.xls for downstream scGene_matrix and scIsoform_matrix matrix building.

## Related tools

- **scISA-Tools (cellBC_UMI_corrector)** (Pipeline module that wraps CellRanger's cellBC/UMI correction logic for long FLNC reads; performs UMI quality filtering, whitelist barcode correction, and writes flnc.cellBC_UMI_correction.xls) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger 3.1.0** (Source of the 10x barcode whitelist (3M-february-2018.txt) and the barcode/UMI correction strategy reimplemented in cellBC_UMI_corrector) — https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- **classify_by_primer** (Upstream Perl utility that extracts cell barcodes and UMIs and produces isoseq_flnc.BarcodeUMI.fastq from primer-classified CCS reads) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- flnc.cellBC_UMI_correction.xls exists and is populated
- Every emitted cell barcode in the corrected table is a member of the 3M-february-2018.txt whitelist (or the read was dropped)
- Number of corrected records is consistent with the number of FLNC entries in isoseq_flnc.BarcodeUMI.fastq passing the UMI quality filters
- No retained UMI is a homopolymer, contains N, or has any base with base quality <10
- Uncorrectable barcodes (nearest whitelist match >1 Hamming distance or posterior ≤0.975) are excluded, not assigned arbitrarily

## Limitations

- Barcodes/UMIs from reads with bases below base quality 10, homopolymer UMIs, or N-containing UMIs are discarded, so noisy molecules are lost rather than rescued
- Correction relies on the 10x Genomics whitelist (3M-february-2018.txt); barcodes absent from the whitelist cannot be recovered, and the approach assumes a 10x Genomics-style capture
- The workflow was demonstrated only with the 10x Genomics platform in HIT-scISOseq; compatibility with other single-cell platforms is claimed but not shown

## Evidence

- [methods] cellBC_UMI_corrector wraps CellRanger correction: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [discussion] UMI quality filter criteria: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] Barcode correction posterior threshold: [evidence span withheld — anchor status: not_found]
- [readme] BarcodeUMI fastq produced by classify_by_primer: "will find three file in your specified output path: `isoseq_flnc.BarcodeUMI.fastq`, `isoseq_flnc.Transcript.fastq`, `isoseq.PrimerStat.csv`"
- [readme] Correction needs no short-read guidance data: "these methods using long reads data independently, no need additional short read data for guiding"
