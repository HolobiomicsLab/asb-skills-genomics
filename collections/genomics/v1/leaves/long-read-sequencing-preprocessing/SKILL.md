---
name: long-read-sequencing-preprocessing
description: Use when preprocessing nanopore (PromethION) FASTQ reads from enzymatic
  methyl-seq (EMseq) data for alignment, where unmethylated cytosines were converted
  to thymine and a C-to-T / G-to-A dual-strand reference conversion is required.
when_to_use_negative:
- Reads are native (unconverted) nanopore WGS to be analyzed directly with nanopolish
  signal-level methylation calling — no base conversion preprocessing is needed or
  appropriate.
- Data are short-read bisulfite/EMseq reads for standard bismark alignment, which
  has its own conversion-aware aligner and does not need this nanopore-oriented FASTQ
  conversion script.
- Input is already base-converted FASTQ (*_CT.fq.gz or *_GA.fq.gz) — re-running the
  conversion would double-substitute bases.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: convert_reads.py
  role: Python3/pysam script that performs C-to-T conversion of the read sequences
    and emits the companion G-to-A converted FASTQ
  repo: https://github.com/yos-sk/nanoEM
- name: convert_ref.py
  role: Script that converts all C/Gs in the reference genome to T/As, producing output.fa
    used for downstream minimap2 alignment
  repo: https://github.com/yos-sk/nanoEM
- name: pysam
  role: Python library used to read and write FASTQ records during base conversion
- name: Python3
  role: Runtime for the conversion scripts in the nanoEM pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:538f7532383673ae2452273af11d9280d6d1fb02ac44575a73ae49bf9f1f9fc9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - convert_reads.py
  - convert_ref.py
  - pysam
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# long-read-sequencing-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the nanoEM preprocessing step: bismark-like base conversion of both the nanopore FASTQ reads and the reference genome so that EMseq-converted reads (unmethylated C read as T) can be aligned correctly, producing paired *_CT.fq.gz and *_GA.fq.gz read sets and a C-to-T / G-to-A converted reference FASTA.

## When to use

Use when preprocessing nanopore (PromethION) FASTQ reads from enzymatic methyl-seq (EMseq) data for alignment, where unmethylated cytosines were converted to thymine and a C-to-T / G-to-A dual-strand reference conversion is required.

## When NOT to use

- Reads are native (unconverted) nanopore WGS to be analyzed directly with nanopolish signal-level methylation calling — no base conversion preprocessing is needed or appropriate.
- Data are short-read bisulfite/EMseq reads for standard bismark alignment, which has its own conversion-aware aligner and does not need this nanopore-oriented FASTQ conversion script.
- Input is already base-converted FASTQ (*_CT.fq.gz or *_GA.fq.gz) — re-running the conversion would double-substitute bases.

## Inputs

- Raw nanoEM nanopore sequencing FASTQ (PromethION 1D pass reads, 3.4–7.6 kb N50)
- Reference genome FASTA (passed as 'ref' to the conversion script)

## Outputs

- *_CT.fq.gz: C-to-T converted gzipped FASTQ
- *_GA.fq.gz: G-to-A converted gzipped FASTQ
- output.fa: C-to-T converted reference genome FASTA

## How to apply

Load the raw nanoEM nanopore FASTQ with src/convert_reads.py (Python3 + pysam) and perform a C-to-T substitution on read sequences, mimicking EMseq enzymatic conversion where unmethylated cytosines are read as thymine; simultaneously emit the companion G-to-A converted FASTQ representing the opposite strand orientation, writing both as gzip-compressed FASTQ files (*_CT.fq.gz and *_GA.fq.gz). Separately convert the reference genome with src/convert_ref.py (all C/Gs to T/As) to produce output.fa for downstream minimap2 alignment. The dual CT/GA representation is required because minimap2 alignment of each converted read set against the converted reference must be combined later via best_align.py to choose the best alignment per read. Validate that the converted files exist as gzipped FASTQ, contain the same read set as the input, and show C-to-T (and G-to-A) substitution relative to the input sequences.

## Related tools

- **convert_reads.py** (Python3/pysam script that performs C-to-T conversion of the read sequences and emits the companion G-to-A converted FASTQ) — https://github.com/yos-sk/nanoEM
- **convert_ref.py** (Script that converts all C/Gs in the reference genome to T/As, producing output.fa used for downstream minimap2 alignment) — https://github.com/yos-sk/nanoEM
- **pysam** (Python library used to read and write FASTQ records during base conversion)
- **Python3** (Runtime for the conversion scripts in the nanoEM pipeline)

## Examples

```
python src/convert_reads.py fastq && python src/convert_ref.py ref > output.fa
```

## Evaluation signals

- Both *_CT.fq.gz and *_GA.fq.gz exist and are valid gzip-compressed FASTQ files.
- The converted FASTQ contains exactly the same read set (names and count) as the input FASTQ.
- Sequence content of *_CT.fq.gz shows C-to-T substitution (and *_GA.fq.gz shows G-to-A substitution) relative to the input reads, with no other base changes.
- The converted reference output.fa reflects C/G-to-T/A substitution genome-wide and aligns cleanly with the converted reads via minimap2 --eqx in the subsequent mapping step.

## Limitations

- No software version numbers are stated for the conversion scripts or dependencies in the README, so reproducibility depends on the repository state at time of use.
- The conversion is deterministic on base identities only — it cannot distinguish methylated from unmethylated cytosines; methylation is inferred later from alignment discrepancies.
- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, a limitation inherited by all downstream analysis of the converted reads.

## Evidence

- [methods] The pipeline applies a bismark-like base conversion: all C/Gs in the human reference genome are converted to T/As, and all C/Gs in the sequencing reads are converted to G/As prior to alignment.: [evidence span withheld — anchor status: not_found]
