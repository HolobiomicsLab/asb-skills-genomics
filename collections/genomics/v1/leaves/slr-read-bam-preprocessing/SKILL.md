---
name: slr-read-bam-preprocessing
description: Use when running Blackbird on 10X/stLFR synthetic long read data (e.g.
  HG002 NA24385 GRCh37 phased possorted BAM) or adding PacBio HiFi reads in hybrid
  mode, and the BAM/reference inputs must first be sorted, indexed, and barcode-tagged.
when_to_use_negative:
- Input reads are plain short-read Illumina data without BX barcode tags, which Blackbird's
  SLR mode cannot use for barcode-aware assembly
- The goal is SV calling directly from long reads alone (e.g. only PacBio HiFi) —
  Sniffles2 or PBSV are the appropriate callers, not Blackbird SLR preprocessing
- The BAM is unsorted or unindexed and downstream tools other than Blackbird, which
  do not require BWA-indexed references, will consume it
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: SV caller consuming the preprocessed SLR BAM, BWA-indexed reference, and optional
    long-read BAM/FASTQ pair
  repo: https://github.com/1dayac/Blackbird
- name: BWA
  role: Indexes the reference genome as required by Blackbird's --rerefence option
- name: SPAdes
  role: Forked codebase (spades_for_blackbird) compiled via spades_compile.sh to provide
    the local-assembly backend
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:9d5709516248585db8ddaf294da84de484ff5f5da0dc2daac1986c114363fc75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - Manta
  - PBSV
  - SPAdes
  - Sniffles2
  - SvABA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# slr-read-bam-preprocessing

## Summary

Prepare synthetic long read (SLR) inputs for Blackbird structural variant calling: a position-sorted, indexed SLR BAM carrying BX barcode tags plus a BWA-indexed reference genome, optionally paired with sorted/indexed long-read BAM and FASTQ for hybrid mode.

## When to use

Use when running Blackbird on 10X/stLFR synthetic long read data (e.g. HG002 NA24385 GRCh37 phased possorted BAM) or adding PacBio HiFi reads in hybrid mode, and the BAM/reference inputs must first be sorted, indexed, and barcode-tagged.

## When NOT to use

- Input reads are plain short-read Illumina data without BX barcode tags, which Blackbird's SLR mode cannot use for barcode-aware assembly
- The goal is SV calling directly from long reads alone (e.g. only PacBio HiFi) — Sniffles2 or PBSV are the appropriate callers, not Blackbird SLR preprocessing
- The BAM is unsorted or unindexed and downstream tools other than Blackbird, which do not require BWA-indexed references, will consume it

## Inputs

- Position-sorted and indexed SLR read BAM with BX tags (e.g. NA24385.GRCh37.phased_possorted_bam.bam)
- BWA-indexed reference genome FASTA (GRCh37/hg19)
- Optional: position-sorted and indexed long-read BAM paired with a FASTQ of the same long reads (e.g. PacBio HiFi 34x HG002)

## Outputs

- Validated SLR BAM index and BWA reference index ready for Blackbird --bam/--rerefence arguments
- Prepared long-read BAM/FASTQ pair ready for Blackbird --long-bam/--long-read arguments in hybrid mode

## How to apply

Confirm the SLR BAM (e.g. NA24385.GRCh37.phased_possorted_bam.bam on GRCh37/hg19) is position-sorted and indexed and carries BX barcode tags, since Blackbird relies on barcode-aware sliding-window assembly. Index the reference genome with BWA because Blackbird requires a BWA-indexed reference via --rerefence/-r. If using hybrid mode, prepare a position-sorted and indexed long-read BAM (--long-bam/-l) paired with a FASTQ of the same long reads (--long-read/-m); these two options must always be supplied together. Also install/compile the SPAdes-for-Blackbird fork and run spades_compile.sh so the blackbird binary and spades.py are available for the local-assembly stage.

## Related tools

- **Blackbird** (SV caller consuming the preprocessed SLR BAM, BWA-indexed reference, and optional long-read BAM/FASTQ pair) — https://github.com/1dayac/Blackbird
- **BWA** (Indexes the reference genome as required by Blackbird's --rerefence option)
- **SPAdes** (Forked codebase (spades_for_blackbird) compiled via spades_compile.sh to provide the local-assembly backend) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
bwa index genome.fa && samtools index NA24385.GRCh37.phased_possorted_bam.bam
```

## Evaluation signals

- SLR BAM is position-sorted with a valid .bai index and reads carry BX barcode tags
- Reference genome directory contains a complete BWA index set before invoking Blackbird
- In hybrid mode, --long-bam and --long-read are both supplied and the long-read BAM is sorted and indexed
- Blackbird run completes and produces the position-sorted out_50.vcf with insertion/deletion calls longer than 50 bp
- Runtime and peak memory (e.g. Blackbird SLR 6 h / 12 Gb; hybrid 11 h / 117 Gb on HG002 under /usr/bin/time -v) are within hardware-dependent tolerance of reported values

## Limitations

- Blackbird requires BX barcode tags on the SLR BAM; datasets without barcodes are unsupported
- The long-read BAM and FASTQ are optional only as a paired set — one without the other is invalid
- The reference option flag is misspelled '--rerefence' in the README; commands must use this spelling (or -r) to work

## Evidence

- [readme] SLR BAM must be sorted/indexed with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Reference must be BWA-indexed: "`--rerefence` or `-r` [required] - BWA-indexed reference genome"
- [readme] Hybrid mode pairs long-read BAM and FASTQ: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [methods] Compile via spades_compile.sh: "To compile Blackbird in assembler foder execute:

``n./spades_compile.sh"
- [methods] Barcode-aware sliding window rationale: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
