---
name: long-read-masseq-data-retrieval
description: Use when a benchmark or pipeline evaluation requires real PacBio MAS-Seq
  / Kinnex-Iso-Seq long-read single-cell cDNA reads (e.g. Revio PBMC runs) and no
  local FASTQ/BAM input exists yet.
when_to_use_negative:
- Input data are already deconcatenated segment BAMs or per-gene/per-cell molecule
  count matrices rather than concatenated MAS-Seq reads
- The target platform is ONT amplicon or 10x short-read cDNA libraries rather than
  PacBio MAS-Seq/Kinnex-Iso-Seq single-cell reads
- Only reference transcriptomes or tool installation is missing — no dataset retrieval
  step is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: Execution environment and downstream pipeline that consumes the retrieved
    MAS-Seq reads for deconcatenation and molecule quantification
  repo: github.com/mehdiborji/nanoranger
- name: minimap2
  role: Splice-aware aligner with the PacBio Kinnex/Iso-Seq preset used to map retrieved
    long RNA/cDNA reads against the GENCODE reference
  repo: https://github.com/lh3/minimap2
- name: Pacific Biosciences
  role: Source platform whose MAS-Seq Revio runs (PBMC cDNA sequencing) provide the
    public benchmark data on downloads.pacbcloud.com
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:58471fb1bf5fa33b88f22a29146c32e15f3a43b1119678bd8defe3ddef72aef8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pacific Biosciences
  - minimap2
  - nanoranger
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-masseq-data-retrieval

## Summary

Retrieves public PacBio Revio MAS-Seq (MAS-ISO-seq) long-read single-cell cDNA datasets, such as DATA-Revio-PBMC-2 from downloads.pacbcloud.com, to serve as benchmark input for long-read single-cell pipelines like nanoranger.

## When to use

Use when a benchmark or pipeline evaluation requires real PacBio MAS-Seq / Kinnex-Iso-Seq long-read single-cell cDNA reads (e.g. Revio PBMC runs) and no local FASTQ/BAM input exists yet.

## When NOT to use

- Input data are already deconcatenated segment BAMs or per-gene/per-cell molecule count matrices rather than concatenated MAS-Seq reads
- The target platform is ONT amplicon or 10x short-read cDNA libraries rather than PacBio MAS-Seq/Kinnex-Iso-Seq single-cell reads
- Only reference transcriptomes or tool installation is missing — no dataset retrieval step is needed

## Inputs

- Public PacBio Revio MAS-Seq run identifier (e.g. DATA-Revio-PBMC-2) hosted on downloads.pacbcloud.com
- Host URL downloads.pacbcloud.com with recursive dataset directory listing
- GENCODE v44 transcriptome reference for downstream alignment

## Outputs

- Local copies of raw MAS-Seq long-read sequencing files (FASTQ/BAM) for the PBMC run
- Input read set ready for nanoranger/skera deconcatenation and quantification benchmarks
- Side-by-side benchmark table (e.g. nanoranger_vs_skera_comparison.csv) produced downstream

## How to apply

Identify the target public run (e.g. the PacBio Revio MAS-Seq PBMC run DATA-Revio-PBMC-2) on the PacBio cloud data host downloads.pacbcloud.com and download the read files recursively. Supply the GENCODE v44 transcriptome as the alignment reference when these reads are fed into the nanoranger workflow (github.com/mehdiborji/nanoranger), configuring the TruSeq-adapter barcode search per the Methods. Confirm the retrieved files are the raw long-read cDNA sequencing output (FASTQ/BAM from PacBio devices) rather than pre-processed segment calls, since deconcatenation (e.g. nanoranger vs skera) must start from concatenated MAS-Seq reads. Validate the download by checking file integrity and that read counts/lengths are consistent with multi-segment single-cell cDNA libraries before running quantification.

## Related tools

- **nanoranger** (Execution environment and downstream pipeline that consumes the retrieved MAS-Seq reads for deconcatenation and molecule quantification) — github.com/mehdiborji/nanoranger
- **minimap2** (Splice-aware aligner with the PacBio Kinnex/Iso-Seq preset used to map retrieved long RNA/cDNA reads against the GENCODE reference) — https://github.com/lh3/minimap2
- **Pacific Biosciences** (Source platform whose MAS-Seq Revio runs (PBMC cDNA sequencing) provide the public benchmark data on downloads.pacbcloud.com)

## Examples

```
wget -r -np https://downloads.pacbcloud.com/public/DATA-Revio-PBMC-2/
```

## Evaluation signals

- Downloaded directory contains raw read files for run DATA-Revio-PBMC-2 with plausible MAS-Seq read lengths (multi-kb concatenated reads), not segment-resolved output
- File checksums/completeness verified after recursive retrieval from downloads.pacbcloud.com
- Downstream nanoranger run on the retrieved reads yields per-read segment counts in the expected range (median ~12 segments/read vs skera's 15)
- Resulting molecules-per-gene and molecules-per-cell values correlate with skera reference values (r = 0.88 and 0.97)
- nanoranger_vs_skera_comparison.csv exists, is well-formed CSV, and contains rows for segments-per-read and molecules-per-gene with skera references (12 vs 15; 141 vs 55)

## Limitations

- Segment counts per read differ between tools (nanoranger median 12, range 0–37 vs skera median 15, range 0–16), so downstream counts must not be compared segment-for-segment without noting tool-specific deconcatenation behavior
- Nanoranger is designed for 10x Genomics cDNA library or targeted-amplicon input sequenced on ONT or PacBio devices; other chemistries are outside its validated scope
- Genotyping sensitivity is bounded by transcript representation — low or absent long transcripts cannot be recovered by deeper sequencing of the same input

## Evidence

- [readme] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or Pacific Biosciences devices.: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] minimap2 PacBio Kinnex/Iso-seq preset for spliced alignment of long cDNA reads: [evidence span withheld — anchor status: external_doc]
