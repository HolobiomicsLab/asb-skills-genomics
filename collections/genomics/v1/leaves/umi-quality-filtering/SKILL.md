---
name: umi-quality-filtering
description: Use when processing isoseq_flnc.BarcodeUMI.fastq files from HIT-scISOseq
  data and UMIs must be validated against homopolymer, N-base, and base-quality criteria
  before cellBC correction and UMI collapsing.
when_to_use_negative:
- Input is raw CCS/subreads BAM that has not yet been classified by classify_by_primer
  to extract cellBC and UMI sequences
- Short-read 10x Gene Expression data already processed by CellRanger, which applies
  its own barcode/UMI correction natively
- UMIs carry no per-base quality scores, since the base quality <10 rule cannot be
  evaluated
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides the cellBC_UMI_corrector module that implements UMI quality filtering,
    cell barcode correction against the whitelist, and UMI collapsing
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: cellBC_UMI_corrector wraps CellRanger's cell barcode and UMI correction function;
    long reads are processed independently without short-read guidance
- name: classify_by_primer
  role: Upstream scISA-Tools utility that produces the isoseq_flnc.BarcodeUMI.fastq
    input containing cell barcode and UMI sequences with base qualities
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:aceb91ecca91aba6a38962c3ba4010754a0b81cf5d553145d7cbc1c25a5d4f0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - classify_by_primer
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# UMI quality filtering

## Summary

A quality-control step in the scISA-Tools cellBC_UMI_corrector module that discards low-confidence UMIs from HIT-scISOseq FLNC reads before barcode/UMI correction and downstream single-cell matrix generation.

## When to use

Use when processing isoseq_flnc.BarcodeUMI.fastq files from HIT-scISOseq data and UMIs must be validated against homopolymer, N-base, and base-quality criteria before cellBC correction and UMI collapsing.

## When NOT to use

- Input is raw CCS/subreads BAM that has not yet been classified by classify_by_primer to extract cellBC and UMI sequences
- Short-read 10x Gene Expression data already processed by CellRanger, which applies its own barcode/UMI correction natively
- UMIs carry no per-base quality scores, since the base quality <10 rule cannot be evaluated

## Inputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads with cell barcode and UMI, with base qualities)
- 3M-february-2018.txt (10x Genomics cell barcode whitelist)
- flnc.filtered.tmap (gffcompare isoform assignment table filtered by filter.tmap.pl)

## Outputs

- flnc.cellBC_UMI_correction.xls (table of FLNC reads with quality-passing, corrected cellBC and collapsed UMI assignments joined to isoform assignments)

## How to apply

After classify_by_primer produces isoseq_flnc.BarcodeUMI.fastq, load it together with the 10x Genomics whitelist (3M-february-2018.txt) and the gffcompare-filtered isoform table (flnc.filtered.tmap) in the cellBC_UMI_corrector module (which wraps CellRanger's barcode/UMI correction). Apply three UMI acceptance rules: reject UMIs that are homopolymers (e.g. AAAAAAAAAA), reject UMIs containing N bases, and reject UMIs with any base whose base quality score is below 10. Reads whose UMI fails these checks are dropped before the observed cell barcode is corrected to the nearest whitelist barcode (Hamming distance ≤1, posterior probability >0.975) and UMIs within 1 Hamming distance are collapsed per gene/cell. This pre-filter prevents sequencing-error-derived or uninformative UMIs from inflating molecule counts when joining corrected cellBC/UMI assignments with tmap isoform assignments to write flnc.cellBC_UMI_correction.xls.

## Related tools

- **scISA-Tools** (Provides the cellBC_UMI_corrector module that implements UMI quality filtering, cell barcode correction against the whitelist, and UMI collapsing) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (cellBC_UMI_corrector wraps CellRanger's cell barcode and UMI correction function; long reads are processed independently without short-read guidance)
- **classify_by_primer** (Upstream scISA-Tools utility that produces the isoseq_flnc.BarcodeUMI.fastq input containing cell barcode and UMI sequences with base qualities) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
perl filter.tmap.pl flnc.isoseq_flnc.Transcript.gff.tmap > flnc.filtered.tmap  # then run scISA-Tools cellBC_UMI_corrector on isoseq_flnc.BarcodeUMI.fastq with 3M-february-2018.txt whitelist
```

## Evaluation signals

- flnc.cellBC_UMI_correction.xls exists after the run
- Every emitted cell barcode is a member of the 3M-february-2018.txt whitelist, or the read is dropped
- No retained UMI is a homopolymer (e.g. AAAAAAAAAA), contains N bases, or has any base with base quality <10
- The number of corrected records is consistent with the number of FLNC entries in the input fastq that pass the quality filters
- UMIs collapsed per gene/cell differ by at most 1 Hamming distance from their merged representative

## Limitations

- Reads whose UMI fails any quality rule are discarded entirely, reducing usable FLNC counts
- Correction depends on the 10x Genomics 3M-february-2018 whitelist; non-10x barcode chemistries are not covered
- CellBC correction requires posterior probability >0.975 within 1 Hamming distance; ambiguous barcodes near threshold may be dropped

## Evidence

- [discussion] UMI quality filtering: no homopolymers, no N bases, no bases with quality <10: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] CellBC correction against whitelist with posterior threshold: [evidence span withheld — anchor status: not_found]
- [methods] cellBC_UMI_corrector wraps CellRanger correction: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [methods] Whitelist and example data provided in repo: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
- [readme] classify_by_primer produces the BarcodeUMI fastq input: "After running finished, will find three file in your specified output path: `isoseq_flnc.BarcodeUMI.fastq`, `isoseq_flnc.Transcript.fastq`, `isoseq.PrimerStat.csv`"
