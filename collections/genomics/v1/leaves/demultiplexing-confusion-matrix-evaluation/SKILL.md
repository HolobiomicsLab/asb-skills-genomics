---
name: demultiplexing-confusion-matrix-evaluation
description: Use when validating a barcode demultiplexing tool against spiked-in SIRV
  FLNC reads with known isoform ground truth, to compute accuracy, specificity, and
  a confusion matrix of assignment errors.
when_to_use_negative:
- Input reads lack a ground-truth identity (e.g., ordinary cellular transcriptomes
  without spike-ins), so a confusion matrix against expected isoforms cannot be built
- The goal is isoform quantification per cell (use scIsoform_matrix) rather than evaluating
  demultiplexing accuracy itself
- Reads are raw CCS reads not yet classified to FLNC with cell barcode and UMI extracted
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: scISA-Tools
  role: Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads
    with cell barcode correction (cellBC_UMI_corrector) against the 10x whitelist;
    the subject of the accuracy evaluation
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Provides the cell barcode and UMI correction logic (3M-february-2018.txt whitelist)
    wrapped as the cellBC_UMI_corrector module
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- name: PacBio Sequel II
  role: Generated the CCS reads from the SMRT Cell 8M that underlie the spiked-in
    SIRV FLNC evaluation data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:38a8dabb0ee16318d2ee36699837d15456bfc4c73b1d43f4decaec84ae0101e2
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# demultiplexing-confusion-matrix-evaluation

## Summary

Benchmarks single-cell barcode demultiplexing accuracy by assigning demultiplexed reads to a ground-truth spike-in identity (SIRV-Set 4 isoforms), building an isoform assignment confusion matrix, and computing accuracy and specificity metrics. This reproduces the scISA-Tools evaluation showing >99.99% accuracy/specificity on HIT-scISOseq data.

## When to use

Use when validating a barcode demultiplexing tool against spiked-in SIRV FLNC reads with known isoform ground truth, to compute accuracy, specificity, and a confusion matrix of assignment errors.

## When NOT to use

- Input reads lack a ground-truth identity (e.g., ordinary cellular transcriptomes without spike-ins), so a confusion matrix against expected isoforms cannot be built
- The goal is isoform quantification per cell (use scIsoform_matrix) rather than evaluating demultiplexing accuracy itself
- Reads are raw CCS reads not yet classified to FLNC with cell barcode and UMI extracted

## Inputs

- Barcoded SIRV-Set 4 (Lexogen) FLNC reads (BarcodeUMI fastq) from the spiked-in condition
- 10x Genomics cell barcode whitelist 3M-february-2018.txt
- SIRV spike-in reference with known isoform identities (ground truth)

## Outputs

- sirv_demux_confusion_matrix.csv (SIRV isoform assignment confusion matrix)
- demux_metrics.csv (demultiplexing accuracy and specificity)

## How to apply

Load the barcoded SIRV-Set 4 FLNC reads (BarcodeUMI fastq from the spiked-in condition) and the 10x Genomics barcode whitelist 3M-february-2018.txt. Run scISA-Tools demultiplexing, applying cell barcode correction against the whitelist with a posterior probability threshold >0.975 and UMI quality filters (no homopolymers, no N bases, no bases with base quality <10). Assign each demultiplexed read to its expected SIRV-Set 4 isoform identity using the known SIRV spike-in reference, then build the isoform assignment confusion matrix comparing demultiplexed assignments against ground truth (corresponding to Fig. 2b-d). Compute overall demultiplexing accuracy and specificity from the confusion matrix; the article reports 99.997% accuracy, 99.998% specificity, and a SIRV isoform assignment confusion rate as low as 0.1066%. Save the confusion matrix as sirv_demux_confusion_matrix.csv and the metrics as demux_metrics.csv.

## Related tools

- **scISA-Tools** (Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads with cell barcode correction (cellBC_UMI_corrector) against the 10x whitelist; the subject of the accuracy evaluation) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Provides the cell barcode and UMI correction logic (3M-february-2018.txt whitelist) wrapped as the cellBC_UMI_corrector module) — https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- **PacBio Sequel II** (Generated the CCS reads from the SMRT Cell 8M that underlie the spiked-in SIRV FLNC evaluation data)

## Examples

```
python cellBC_UMI_corrector.py --input sirv_flnc_barcode_umi.fq --whitelist 3M-february-2018.txt --out sirv_demux/
```

## Evaluation signals

- Recomputed demultiplexing accuracy reaches 99.997% (reported >99.99%)
- Recomputed demultiplexing specificity reaches 99.998% (reported >99.99%)
- SIRV isoform assignment confusion rate is as low as 0.1066%
- Confusion matrix matches Fig. 2b-d of the article
- All assigned barcodes are members of the 3M-february-2018.txt whitelist after correction at posterior probability >0.975

## Limitations

- Requires a spike-in with known isoform ground truth (SIRV-Set 4); cannot assess accuracy on genuine cellular samples where truth is unknown
- Evaluation depends on UMI quality filters (no homopolymers, no N, no base quality <10) and the 0.975 posterior threshold; relaxing them changes the measured accuracy
- scISA-Tools compatibility is only demonstrated with the 10x Genomics platform in this study

## Evidence

- [abstract] scISA-Tools achieved 99.997% and 99.998% barcode assignment accuracy and specificity: "demultiplex HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity"
- [discussion] barcode correction threshold: [evidence span withheld — anchor status: not_found]
- [discussion] UMI quality filters: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] SIRV spike-in dataset: "We prepared cDNA using 1 μL of SIRV-Set 4 (Lexogen) RNA and SuperScriptTM II Reverse Transcriptase"
- [methods] correction module origin: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
