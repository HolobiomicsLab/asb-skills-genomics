---
name: multiplexed-long-read-artifact-omission
description: Use when long reads come from telobait-capture PacBio HiFi data with
  sample barcodes embedded in the telobaits and each read must be assigned to its
  source sample before per-sample telomere quantification.
when_to_use_negative:
- Input is vanilla whole-genome sequencing long-read data with no telobait barcodes
  — run telomap in WGS mode instead (OLIGO and BARCODE will be NA).
- Telobait capture was not performed, so reads carry no embedded sample barcode to
  demultiplex.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: telomap
  role: Demultiplexes telobait-captured reads by aligning each read against capture
    oligo and barcode sequences in telobait mode and omitting multi-mapping artifact
    reads
  repo: https://github.com/cytham/telomap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:4de902a493dfdf5563d7eb7009d7f2366977b9ac17a0bec94b423f59cbfddd6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - telomap
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multiplexed-long-read-artifact-omission

## Summary

Demultiplexes telobait-captured PacBio HiFi telomere reads to individual samples by matching barcodes embedded in the telobaits with telomap (telobait mode), and omits reads that map to more than one capture oligo or barcode as library-preparation artifacts.

## When to use

Use when long reads come from telobait-capture PacBio HiFi data with sample barcodes embedded in the telobaits and each read must be assigned to its source sample before per-sample telomere quantification.

## When NOT to use

- Input is vanilla whole-genome sequencing long-read data with no telobait barcodes — run telomap in WGS mode instead (OLIGO and BARCODE will be NA).
- Telobait capture was not performed, so reads carry no embedded sample barcode to demultiplex.

## Inputs

- Telomeric long reads from PacBio HiFi telobait-capture sequencing (FASTA/FASTQ/BAM)
- Capture oligo FASTA file (capture_oligo.fa; sequence after the barcode on the telobait)
- Sample barcode FASTA file (barcodes.fa; telobait start through end of barcode, one entry per sample)

## Outputs

- sample.telomap.tsv per-read assignment table with RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN columns
- Per-sample read assignment records (barcode, oligo, strand, motif, read length, telomere start/end) enabling per-sample telomere quantification

## How to apply

Run telomap in telobait mode, supplying the telomeric reads (task_001 output), a capture_oligo.fa (sequence after the barcode on the telobait, e.g. GATGCCAGATGCACGGAGCA) and a barcodes.fa listing each sample's telobait prefix-plus-barcode sequence (e.g. >sample1 ATCGACGGTTCAA). Use the default --oligoscore 1 and --barscore 1, i.e. a perfect (fraction 1.0) alignment score for both capture oligo and barcode matches; the tool warns that lowering --barscore can cause multiple barcode mappings per read and high read omission. Reads aligning to more than one capture oligo or more than one barcode are flagged and omitted as telobait library-preparation artifacts. Each passing read is assigned to its detected sample barcode and recorded with OLIGO, BARCODE, STRAND, MOTIF, RLEN, TELOMERE_START/END and TELOMERE_LEN fields in the per-sample read assignment table (sample.telomap.tsv).

## Related tools

- **telomap** (Demultiplexes telobait-captured reads by aligning each read against capture oligo and barcode sequences in telobait mode and omitting multi-mapping artifact reads) — https://github.com/cytham/telomap

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- sample.telomap.tsv exists and contains the OLIGO and BARCODE fields
- No retained read maps to more than one barcode under the required alignment score fraction of 1 (default --barscore 1)
- Every assigned read has a non-NA BARCODE value matching one entry in barcodes.fa
- The proportion of reads omitted for multiple oligo/barcode mappings is low; a high omission rate suggests misconfigured --barscore
- Read-level metadata (STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN) is populated for each demultiplexed read

## Limitations

- Lowering --barscore may lead to multiple barcode mapping per read, causing high read omission (explicit tool warning).
- Telobait mode requires data generated from a wet-lab telomere capturing method; it cannot demultiplex data lacking embedded barcodes.

## Evidence

- [methods] Demultiplex barcoded samples (for telobait-capture data): "De-multiplex barcoded samples (for telobait-capture data)"
- [readme] barscore warning: "Warning: Reducing
this value may lead to multiple barcode mapping
per read, causing high read omission. [1]"
- [intro] barcode embedding rationale: "Embedding unique barcodes within the telobaits allows multiplexing of samples and, in turn, permits high-throughput telomere length measurement at nucleotide resolution"
- [readme] FASTA content definition: "The capture oligo sequence refers to the sequence after the barcode on the telobait, while the barcode sequence refers to the start of the telobait until the end of the barcode."
- [methods] oligoscore definition: "--oligoscore float minimum alignment score fraction required for capture oligo sequence match [1]"
