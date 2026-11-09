---
name: cell-barcode-whitelist-correction
description: Use when demultiplexing HIT-scISOseq FLNC BarcodeUMI fastq reads into
  per-cell cDNA reads and observed 10x cell barcodes must be corrected against a whitelist
  with posterior-probability thresholds.
when_to_use_negative:
- Input reads lack cell barcodes entirely, e.g. TSO-artifact reads already removed
  by the HIT-scISOseq capture step — there is nothing to correct
- Data were generated on a non-10x Genomics platform whose barcode architecture does
  not match the 3M-february-2018 whitelist, since compatibility beyond 10x was not
  demonstrated in the article
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides the cellBC_UMI_corrector module that wraps CellRanger cell barcode
    and UMI correction for demultiplexing HIT-scISOseq concatenated reads
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Source of the whitelist-based cell barcode and UMI correction function reimplemented
    as cellBC_UMI_corrector; CellRanger 3.1.0 is the referenced version
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- name: 10x Genomics
  role: Supplier of the 3M-february-2018.txt barcode whitelist used as the reference
    for barcode correction
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:4de7ba35a39fea44f3c517f1c441607bfc1d58a8b342ac0520dcd73da78f0f54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - PacBio Sequel II
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-barcode-whitelist-correction

## Summary

Correct observed cell barcodes against the 10x Genomics 3M-february-2018 whitelist and apply UMI quality filters using the scISA-Tools cellBC_UMI_corrector module, enabling accurate demultiplexing of HIT-scISOseq concatenated FLNC reads into single-cell cDNA reads with >99.99% accuracy and specificity.

## When to use

Use when demultiplexing HIT-scISOseq FLNC BarcodeUMI fastq reads into per-cell cDNA reads and observed 10x cell barcodes must be corrected against a whitelist with posterior-probability thresholds.

## When NOT to use

- Input reads lack cell barcodes entirely, e.g. TSO-artifact reads already removed by the HIT-scISOseq capture step — there is nothing to correct
- Data were generated on a non-10x Genomics platform whose barcode architecture does not match the 3M-february-2018 whitelist, since compatibility beyond 10x was not demonstrated in the article

## Inputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads with cell barcode and UMI)
- 3M-february-2018.txt (10x Genomics cell barcode whitelist)
- flnc.filtered.tmap (filtered isoform assignment table from filter.tmap.pl)

## Outputs

- flnc.cellBC_UMI_correction.xls (corrected cellBC/UMI assignments joined with isoform assignments)
- demultiplexed single-cell cDNA reads

## How to apply

Load the FLNC barcode/UMI fastq (isoseq_flnc.BarcodeUMI.fastq), the 10x Genomics whitelist (3M-february-2018.txt), and the filtered isoform assignment table (flnc.filtered.tmap). First apply UMI quality filters: discard UMIs that are homopolymers (e.g. AAAAAAAAAA), contain N bases, or contain bases with base quality <10. Then correct each observed cell barcode by matching it to the nearest whitelist barcode within 1 Hamming distance, replacing it with the highest-posterior whitelist barcode only when the posterior probability exceeds 0.975; reads whose barcodes cannot be corrected are dropped, ensuring every emitted barcode is a whitelist member. Collapse UMIs per gene/cell by merging UMIs within 1 Hamming distance to remove sequencing-error-derived duplicates, then join corrected cellBC/UMI assignments with the tmap isoform assignments per FLNC read. The rationale is that this CellRanger-derived correction logic, wrapped as the cellBC_UMI_corrector module in scISA-Tools, achieves 99.997% accuracy and 99.998% specificity on spiked-in SIRV-Set 4 FLNC reads.

## Related tools

- **scISA-Tools** (Provides the cellBC_UMI_corrector module that wraps CellRanger cell barcode and UMI correction for demultiplexing HIT-scISOseq concatenated reads) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Source of the whitelist-based cell barcode and UMI correction function reimplemented as cellBC_UMI_corrector; CellRanger 3.1.0 is the referenced version) — https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- **10x Genomics** (Supplier of the 3M-february-2018.txt barcode whitelist used as the reference for barcode correction)

## Examples

```
python cellBC_UMI_corrector.py --flnc isoseq_flnc.BarcodeUMI.fastq --whitelist 3M-february-2018.txt --tmap flnc.filtered.tmap --out flnc.cellBC_UMI_correction.xls
```

## Evaluation signals

- Every emitted cell barcode in flnc.cellBC_UMI_correction.xls is a member of the 3M-february-2018 whitelist (or the read was dropped)
- Number of corrected records is consistent with the number of FLNC fastq entries passing the UMI quality filters (no homopolymers, no N, no base quality <10)
- On the spiked-in barcoded SIRV-Set 4 FLNC reads, recomputed demultiplexing accuracy and specificity reproduce >99.99% (99.997% accuracy, 99.998% specificity)
- SIRV isoform assignment confusion rate against the SIRV-Set 4 ground truth is as low as 0.1066%, matching Fig. 2b-d
- No corrected barcode arises from a match at Hamming distance >1 or with posterior probability ≤0.975

## Limitations

- The 10x Genomics ScISOr-Seq pipeline introduces ~50% cell-barcode-free reads (mostly TSO artifacts), so upstream TSO-artifact removal is required before correction is meaningful
- Platform compatibility was only demonstrated with 10x Genomics; use with other single-cell platforms is untested
- Nanopore-based reads with higher error rates would suffer low cell-barcode demultiplex efficiency; this correction workflow is tuned to PacBio HiFi/CCS reads (>99.9% consensus accuracy)

## Evidence

- [discussion] Posterior probability threshold >0.975: [evidence span withheld — anchor status: not_found]
- [discussion] UMI quality filter criteria: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [methods] cellBC_UMI_corrector wraps CellRanger logic: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [abstract] >99.99% accuracy and specificity claim: "demultiplex HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity"
- [methods] Whitelist file location: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
