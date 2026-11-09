---
name: minion-long-read-qc-thresholding
description: Use when you have per-sample demultiplexed Nanopore FASTQ files of multi-kilobase
  rDNA amplicons and need to discard low-quality/short reads before consensus calling
  or downstream barcode analysis.
when_to_use_negative:
- Input FASTQ has not yet been demultiplexed by MiniBar — filter per-sample only after
  dual-index binning, not on the raw combined MinION read file
- Sequencing short amplicons (e.g. COI barcodes or Illumina short reads) — the 3000
  bp minimum length cutoff would discard all reads, since it is sized for multi-kilobase
  rDNA amplicons
- Reads still carry barcode and primer sequences at their ends and you have not run
  MiniBar -T trimming — filtering untrimmed reads will not match the reference yield
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoFilt
  role: Performs the quality (>Q13) and length (>3000 bp) filtering of demultiplexed
    reads
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads into per-sample FASTQ files (-F) and
    trims barcodes/primers (-T) to produce the filter input
  repo: https://github.com/calacademy-research/minibar
- name: Python
  role: Short script used to count FASTQ records in the filtered files and sum them
    across samples for validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:b59a6e1ba732f8069fbbfbabf18521bcd789b3d3865fce26a6a330e7c7c231b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - MiniBar
  - NanoFilt
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# MinION long-read QC thresholding

## Summary

Quality- and length-filters MiniBar-demultiplexed Nanopore long rDNA amplicon reads with NanoFilt, using a minimum average Phred quality of 13 and a minimum read length of 3000 bp, yielding ~245,433 filtered reads for the main run.

## When to use

Use when you have per-sample demultiplexed Nanopore FASTQ files of multi-kilobase rDNA amplicons and need to discard low-quality/short reads before consensus calling or downstream barcode analysis.

## When NOT to use

- Input FASTQ has not yet been demultiplexed by MiniBar — filter per-sample only after dual-index binning, not on the raw combined MinION read file
- Sequencing short amplicons (e.g. COI barcodes or Illumina short reads) — the 3000 bp minimum length cutoff would discard all reads, since it is sized for multi-kilobase rDNA amplicons
- Reads still carry barcode and primer sequences at their ends and you have not run MiniBar -T trimming — filtering untrimmed reads will not match the reference yield

## Inputs

- Per-sample demultiplexed FASTQ files produced by MiniBar -F binning (unassigned reads in the *_unk file), with barcode/primer sequences trimmed via MiniBar -T
- NanoFilt filter parameters: -q 13 (minimum average Phred quality), -l 3000 (minimum read length in bp)

## Outputs

- One quality-filtered FASTQ file per sample containing only reads with average Phred >13 and length >3000 bp
- Per-sample and summed read counts (main run total expected: 245,433 reads)

## How to apply

After demultiplexing dual-indexed MinION reads with MiniBar (using -F to bin reads into per-sample files, ideally with -T so barcodes and primers are trimmed from sequence ends), pipe each sample FASTQ through NanoFilt with -q 13 to require a minimum average Phred quality of 13 and -l 3000 to require a minimum read length of 3000 bp, matching the multi-kilobase rDNA amplicon design. Write the retained reads for each sample to a quality-filtered FASTQ file, then count FASTQ records per sample (e.g. with a short Python script counting every fourth line) and sum across samples. As a validation anchor, the summed filtered read count for the main Nanopore run should be 245,433 reads; substantially fewer suggests over-stringent filtering or a demultiplexing problem upstream.

## Related tools

- **NanoFilt** (Performs the quality (>Q13) and length (>3000 bp) filtering of demultiplexed reads)
- **MiniBar** (Demultiplexes dual-indexed MinION reads into per-sample FASTQ files (-F) and trims barcodes/primers (-T) to produce the filter input) — https://github.com/calacademy-research/minibar
- **Python** (Short script used to count FASTQ records in the filtered files and sum them across samples for validation)

## Examples

```
gunzip -c sample_1.fastq.gz | NanoFilt -q 13 -l 3000 > sample_1.filtered.fastq
```

## Evaluation signals

- Every retained read has average Phred quality >= 13 and sequence length >= 3000 bp (re-checkable directly from the filtered FASTQ)
- Summed filtered read count across samples equals the reported 245,433 reads for the main Nanopore run
- Filtered FASTQ remains valid 4-line FASTQ (each header line followed by sequence, +, qualities of equal length)
- Read counts per sample are non-zero for expected samples; only the *_unk bin is allowed to be residual/discarded
- No retained read begins or ends with barcode/primer sequence, confirming MiniBar -T trimming preceded filtering

## Limitations

- Thresholds are tuned to this pipeline's long rDNA amplicons; Nanopore raw read accuracy is modest, so Q13 is a permissive quality floor appropriate to consensus-based downstream processing rather than per-read accuracy claims
- The 245,433-read yield is specific to the main run; other runs will differ, so this number is a validation anchor only for that dataset
- Filtering alone does not address the taxonomic recovery bias the authors observed in bulk-sample metabarcoding, which they state requires further research

## Evidence

- [other] The pipeline filters demultiplexed reads with Nanofilt using a quality threshold of >13 and a size threshold of >3 kb, and after this quality filtering and trimming the main Nanopore run yielded 245,433 reads: [evidence span withheld — anchor status: not_found]
- [readme] -T trims barcode and primer from each end of the sequence, then outputs record: "-T trims barcode and primer from each end of the sequence, then outputs record"
- [readme] -F create individual sample files for sequences with -S or -C output (default: False): "-F create individual sample files for sequences with -S or -C output (default: False)"
