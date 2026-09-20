---
name: hamming-distance-sequence-matching
description: Use when observed cell barcodes or UMIs from long-read scRNA-seq (e.g.
  HIT-scISOseq FLNC reads) contain sequencing errors and must be corrected against
  a whitelist (e.g. 3M-february-2018.txt) before read-to-cell assignment.
when_to_use_negative:
- Input reads carry no cell barcodes/UMIs (e.g. bulk Iso-Seq FLNC reads or the ~50%
  cell-barcode-free TSO-artifact reads from the 10x ScISOr-Seq pipeline)
- Short-read data with a dedicated 10x CellRanger run is available and preferred,
  since cellBC_UMI_corrector is only needed when guiding short-read data is absent
- Barcodes differ from the 10x Genomics 3M-february-2018 whitelist design and a different
  error model or distance metric is required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scISA-Tools
  role: Provides the cellBC_UMI_corrector module that performs Hamming-distance barcode/UMI
    correction and joins results with the tmap isoform assignments
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Source of the cell barcode and UMI correction logic wrapped by cellBC_UMI_corrector
    (CellRanger 3.1.0)
- name: 10x Genomics whitelist
  role: Supplies the reference barcode list (3M-february-2018.txt) that observed cell
    barcodes are matched against
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:a22d400819202d0fde780e0df52014d95423a9c639f0f87bab9a075660097667
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - 10x Genomics whitelist
  - CellRanger
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Hamming-distance sequence matching

## Summary

Error-tolerant matching of observed cell barcodes and UMIs to a reference whitelist by Hamming distance, as implemented in scISA-Tools' cellBC_UMI_corrector (wrapping CellRanger logic) for HIT-scISOseq FLNC reads.

## When to use

Use when observed cell barcodes or UMIs from long-read scRNA-seq (e.g. HIT-scISOseq FLNC reads) contain sequencing errors and must be corrected against a whitelist (e.g. 3M-february-2018.txt) before read-to-cell assignment.

## When NOT to use

- Input reads carry no cell barcodes/UMIs (e.g. bulk Iso-Seq FLNC reads or the ~50% cell-barcode-free TSO-artifact reads from the 10x ScISOr-Seq pipeline)
- Short-read data with a dedicated 10x CellRanger run is available and preferred, since cellBC_UMI_corrector is only needed when guiding short-read data is absent
- Barcodes differ from the 10x Genomics 3M-february-2018 whitelist design and a different error model or distance metric is required

## Inputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads with extracted cell barcodes and UMIs plus base qualities)
- 10x Genomics whitelist barcode list 3M-february-2018.txt
- flnc.filtered.tmap (filtered isoform assignment table from gffcompare)

## Outputs

- flnc.cellBC_UMI_correction.xls (corrected cellBC/UMI assignments joined to isoform assignments per FLNC read)

## How to apply

First apply UMI quality filtering: discard UMIs that are homopolymers (e.g. AAAAAAAAAA), contain N bases, or contain bases with base quality <10. Correct each observed cell barcode by matching it to the nearest barcode in the 10x Genomics whitelist (3M-february-2018.txt) within 1 Hamming distance, replacing the observed sequence with the highest-posterior whitelist barcode only when the posterior probability exceeds 0.975; otherwise drop the read. Then collapse UMIs per gene/cell by merging UMIs within 1 Hamming distance to remove sequencing-error-derived duplicates. Join the corrected cellBC/UMI assignments with the flnc.filtered.tmap isoform assignments per FLNC read and write flnc.cellBC_UMI_correction.xls. The strategy wraps CellRanger 3.1.0 barcode/UMI correction logic and uses long-read data independently, with no guiding short-read data required.

## Related tools

- **scISA-Tools** (Provides the cellBC_UMI_corrector module that performs Hamming-distance barcode/UMI correction and joins results with the tmap isoform assignments) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Source of the cell barcode and UMI correction logic wrapped by cellBC_UMI_corrector (CellRanger 3.1.0))
- **10x Genomics whitelist** (Supplies the reference barcode list (3M-february-2018.txt) that observed cell barcodes are matched against)

## Examples

```
/path/cellranger-3.1.0/miniconda-cr-cs/4.3.21-miniconda-cr-cs-c10/bin/python cellBC_UMI_corrector --bc_umi_fq isoseq_flnc.BarcodeUMI.fastq --whitelist 3M-february-2018.txt --tmap flnc.filtered.tmap
```

## Evaluation signals

- flnc.cellBC_UMI_correction.xls exists and every emitted cell barcode is a member of the 3M-february-2018.txt whitelist (or the read was dropped)
- UMIs passing filters are non-homopolymer, contain no N bases, and have no bases with base quality <10
- The number of corrected records is consistent with the number of FLNC entries in isoseq_flnc.BarcodeUMI.fastq passing quality filters
- Corrected barcode counts are plausible given a posterior probability threshold of 0.975 (uncertain matches are dropped, not forced)
- Downstream scGene_matrix / scIsoform_matrix counts are reproducible per cell and gene from the corrected assignments

## Limitations

- Correction is capped at 1 Hamming distance with a posterior probability cutoff of 0.975, so reads with >1 barcode error or low posterior are discarded rather than corrected
- The method wraps CellRanger logic designed for the 10x Genomics barcode whitelist; compatibility with other single-cell platforms is claimed but not demonstrated
- UMI collapse within 1 Hamming distance may merge genuinely distinct UMIs at very high sequencing depth (collision risk inherent to distance-1 deduplication)

## Evidence

- [methods] cellBC_UMI_corrector wraps CellRanger correction: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [discussion] posterior threshold 0.975: [evidence span withheld — anchor status: not_found]
- [discussion] UMI quality filters: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [methods] whitelist file source: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
- [readme] no short-read guidance needed: "these methods using long reads data independently, no need additional short read data for guiding."
