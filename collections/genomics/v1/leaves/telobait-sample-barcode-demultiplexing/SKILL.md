---
name: telobait-sample-barcode-demultiplexing
description: Use when long reads come from telobait-capture libraries with sample
  barcodes embedded in the telobaits, and reads must be assigned to their source sample
  for per-sample telomere length quantification.
when_to_use_negative:
- Input reads are vanilla whole-genome sequencing (WGS) long reads with no telobait
  capture barcodes — use telomap's WGS mode instead, which reports NA for OLIGO and
  BARCODE.
- Data were not generated with the wet-lab telobait capture method, so no capture_oligo.fa
  or barcodes.fa are available.
- Input is already-demultiplexed, per-sample read sets where barcode assignment has
  already been performed.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: telomap
  role: Demultiplexes telobait-captured reads by matching embedded capture-oligo and
    sample barcode sequences in telobait mode and writes per-read sample assignment
    metadata
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: Companion PacBio minimap2 wrapper available for alignment of native PacBio
    HiFi reads within the same long-read workflow
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:d515ab3348ef8ebd3758e5b92d3e86f584eaecee6926f5a593f76cf408cb80f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# telobait-sample-barcode-demultiplexing

## Summary

Demultiplexes telobait-captured PacBio HiFi telomere sequencing reads to individual samples (e.g., patient PBL samples) using telomap in telobait mode, which matches the sample-specific barcodes embedded in the telobaits so each sample can be quantified separately.

## When to use

Use when long reads come from telobait-capture libraries with sample barcodes embedded in the telobaits, and reads must be assigned to their source sample for per-sample telomere length quantification.

## When NOT to use

- Input reads are vanilla whole-genome sequencing (WGS) long reads with no telobait capture barcodes — use telomap's WGS mode instead, which reports NA for OLIGO and BARCODE.
- Data were not generated with the wet-lab telobait capture method, so no capture_oligo.fa or barcodes.fa are available.
- Input is already-demultiplexed, per-sample read sets where barcode assignment has already been performed.

## Inputs

- Telomere-containing long reads from telobait-capture data (FASTA/FASTQ/BAM/PacBio-BAM, e.g. reads.fastq.gz)
- Capture oligo FASTA file (capture_oligo.fa; telobait sequence after the barcode)
- Sample barcode FASTA file (barcodes.fa; telobait start through end of barcode, e.g. >sample1 ATCGACGGTTCAA)

## Outputs

- sample.telomap.tsv per-read assignment table with OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN fields
- Per-sample demultiplexed read assignments for downstream telomere length quantification

## How to apply

Run telomap in telobait mode, supplying the telomeric long reads (fastq/fastq.gz/bam) plus the capture oligo FASTA (capture_oligo.fa, the telobait sequence after the barcode) and the sample barcode FASTA (barcodes.fa). Use default --oligoscore 1 and --barscore 1, meaning a perfect alignment score fraction of 1.0 is required for both capture oligo and barcode matches; this strictness avoids telobait library-preparation artifacts. Reads aligning to more than one capture oligo or more than one barcode are flagged and omitted (the README warns that lowering --barscore can cause multiple barcode mappings per read and high read omission). Each passing read is assigned to its detected sample barcode and recorded with read-level metadata (OLIGO, BARCODE, STRAND, MOTIF, RLEN, TELOMERE_START/END) in the per-sample read assignment table.

## Related tools

- **telomap** (Demultiplexes telobait-captured reads by matching embedded capture-oligo and sample barcode sequences in telobait mode and writes per-read sample assignment metadata) — https://github.com/cytham/telomap
- **pbmm2** (Companion PacBio minimap2 wrapper available for alignment of native PacBio HiFi reads within the same long-read workflow) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- sample.telomap.tsv output file exists and contains the OLIGO and BARCODE columns populated for telobait mode.
- No retained read maps to more than one barcode or capture oligo under the required alignment score fraction of 1 (--barscore 1, --oligoscore 1).
- Detected BARCODE values correspond exactly to entries in the supplied barcodes.fa (e.g., sample1, sample2, sample3).
- Proportion of omitted reads due to multiple barcode mappings remains low, consistent with the default --barscore 1 setting.

## Limitations

- Lowering --barscore may lead to multiple barcode mapping per read, causing high read omission.
- Telobait mode is only applicable to data generated by the wet-lab telomere capture method; OLIGO and BARCODE fields will be NA in WGS mode.
- Library-preparation artifacts can cause reads to align to more than one capture oligo or barcode; such reads must be flagged and omitted.

## Evidence

- [methods] Demultiplex barcoded samples (for telobait-capture data): "De-multiplex barcoded samples (for telobait-capture data)"
- [intro] Embedding unique barcodes within the telobaits allows multiplexing: "Embedding unique barcodes within the telobaits allows multiplexing of samples and, in turn, permits high-throughput telomere length measurement at nucleotide resolution"
- [readme] README --barscore option description: "minimum alignment score fraction for barcode sequence match. Warning: Reducing this value may lead to multiple barcode mapping per read, causing high read omission. [1]"
- [readme] README barcode FASTA example: "The barcodes.fa should contain the following sequences >sample1 ATCGACGGTTCAA >sample2 ATCGAGCTGGATT >sample3 ATCGATAACTCGG"
