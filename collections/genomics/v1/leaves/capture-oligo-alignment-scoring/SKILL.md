---
name: capture-oligo-alignment-scoring
description: Use when demultiplexing and identifying telomere-containing reads from
  telobait-captured long-read sequencing (fastq/fasta/bam/pacbio-bam), requiring strict
  capture-oligo and barcode matching before telomere length or TVS analysis.
when_to_use_negative:
- Input reads come from vanilla whole-genome sequencing without telobait capture —
  use telomap WGS mode instead, where OLIGO and BARCODE fields are NA.
- Measuring telomere length in ALT cells whose very long telomeres exceed PacBio HiFi
  DNA polymerase processivity.
- You need unambiguous chromosome-end assignment from short subtelomeric regions (e.g.
  after RsaI/HinfI trimming) — capture-oligo scoring identifies telomeric reads but
  does not resolve specific chromosome ends.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Reads long-read records, performs Bio.Align pairwise alignment of the capture
    oligo (--oligoscore 1) and barcode (--barscore 1) against read ends, scans for
    consecutive TTAGGG repeats, and writes sample.telomap.tsv.
  repo: https://github.com/cytham/telomap
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:bb6887e45ef5bbd8e2d686c0a7bd388a7b0470995e45676ae4f9b75fb5793634
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - telomap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# capture-oligo-alignment-scoring

## Summary

Identifies telomere-containing long reads in telobait-capture PacBio HiFi data by pairwise-aligning the telobait capture oligo (and embedded sample barcode) against read ends with telomap, requiring perfect alignment score fractions, then scanning for at least two consecutive canonical TTAGGG repeats to anchor TELOMERE_START.

## When to use

Use when demultiplexing and identifying telomere-containing reads from telobait-captured long-read sequencing (fastq/fasta/bam/pacbio-bam), requiring strict capture-oligo and barcode matching before telomere length or TVS analysis.

## When NOT to use

- Input reads come from vanilla whole-genome sequencing without telobait capture — use telomap WGS mode instead, where OLIGO and BARCODE fields are NA.
- Measuring telomere length in ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase processivity.
- You need unambiguous chromosome-end assignment from short subtelomeric regions (e.g. after RsaI/HinfI trimming) — capture-oligo scoring identifies telomeric reads but does not resolve specific chromosome ends.

## Inputs

- Long-read sequencing file (fasta/fastq/bam/pacbio-bam) from telobait-capture PacBio HiFi runs
- capture_oligo.fa — capture oligo sequence (telobait sequence after the sample barcode)
- barcodes.fa — sample barcode sequences (start of telobait through end of barcode)

## Outputs

- sample.telomap.tsv — per-read table with RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END
- Retained telomere-containing reads assigned to sample barcodes for demultiplexed quantification

## How to apply

Run telomap in telobait mode, supplying the reads file plus capture_oligo.fa (sequence after the sample barcode on the telobait) and barcodes.fa (start of the telobait through the barcode). Using Bio.Align pairwise alignment, match the capture oligo against the terminal 150 bases at each read end with --oligoscore 1 and the sample barcode with --barscore 1, i.e. both require a perfect (fraction 1.0) alignment score; flag and omit reads aligning to more than one oligo or barcode as library-preparation artifacts. Adjacent to the confirmed capture-oligo site, scan for at least two consecutive canonical telomeric repeats (5'-TTAGGGTTAGGG-3', default motif TTAGGG) and record the read strand and TELOMERE_START, the 1-based 5' position of the telomeric region. Emit one row per retained read (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_START/END) to sample.telomap.tsv for downstream demultiplexing and TVS analysis.

## Related tools

- **telomap** (Reads long-read records, performs Bio.Align pairwise alignment of the capture oligo (--oligoscore 1) and barcode (--barscore 1) against read ends, scans for consecutive TTAGGG repeats, and writes sample.telomap.tsv.) — https://github.com/cytham/telomap

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- Every retained row in sample.telomap.tsv shows a perfect capture-oligo match (oligoscore = 1) at a read end and a barcode match (barscore = 1).
- Each retained read contains a detectable telomeric tract of at least two consecutive TTAGGG repeats (5'-TTAGGGTTAGGG-3') with TELOMERE_START populated.
- All required table fields (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_START/END) are present and non-empty for telobait-mode rows.
- No retained read maps to more than one capture oligo or more than one barcode; high read omission signals artifacts or an inappropriately lowered --barscore.
- OLIGO and BARCODE columns are populated (not NA), distinguishing telobait mode output from WGS mode.

## Limitations

- Lowering --barscore may lead to multiple barcode mappings per read, causing high read omission of telobait library artifacts and ambiguous sample assignment.
- TELOMERE_START requires at least two consecutive canonical repeats, so reads with a single TTAGGG repeat or heavily TVS-disrupted tracts adjacent to the capture site may be missed.
- Restriction-enzyme trimming choices (e.g. HphI, MnlI cutting within TVSs) upstream can bias which telomere-containing fragments survive capture and sequencing.
- Method is incompatible with ALT cells whose very long telomeres exceed PacBio HiFi polymerase processivity.

## Evidence

- [methods] TELOMERE_START definition requiring at least two consecutive telomeric repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] oligoscore threshold: "--oligoscore float minimum alignment score fraction required for capture oligo sequence match [1]"
- [readme] barscore threshold and artifact warning: "minimum alignment score fraction required
                        for barcode sequence match. Warning: Reducing
                        this value may lead to multiple barcode mapping"
- [readme] telomap purpose: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [readme] telobait definition for FASTA construction: "The capture oligo sequence refers to the sequence after the barcode on the telobait, while the barcode sequence refers to the start of the telobait until the end of the barcode."
