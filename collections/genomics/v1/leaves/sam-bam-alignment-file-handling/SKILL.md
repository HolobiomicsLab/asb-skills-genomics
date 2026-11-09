---
name: sam-bam-alignment-file-handling
description: Use when minimap2 has just produced raw SAM alignments of base-called
  ONT direct RNA reads (e.g., curlcake FASTQ vs. synthetic references) and sorted,
  per-condition BAM files are needed for downstream per-site feature extraction.
when_to_use_negative:
- Input is already a sorted BAM or feature table — re-converting adds no information
  and risks losing the mapping state
- Alignments come from a non-ONT platform or a different aligner preset, where '-ax
  map-ont' assumptions do not hold
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: minimap2
  role: Aligns filtered ONT direct RNA reads to the four synthetic references using
    the '-ax map-ont' preset, producing the SAM that this skill converts
- name: samtools
  role: Converts minimap2 SAM output to BAM format and sorts it for downstream per-site
    error-feature extraction
- name: NanoFilt
  role: Upstream quality filtering and headcrop/tailcrop trimming of base-called FASTQ
    reads before alignment
- name: EpiNano
  role: Downstream consumer of the sorted BAMs — extracts per-kmer base-calling error
    features (mismatch, deletion, insertion, quality) for m6A prediction
  repo: https://github.com/novoalab/EpiNano
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:e6ce54e30348855a885e36b37610e6b649253eca58da33e4ae651e4c4d9551e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - NanoFilt
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sam-bam-alignment-file-handling

## Summary

Convert and sort minimap2 SAM alignments of Oxford Nanopore direct RNA reads into indexed BAM files with samtools, so that per-site base-called error features (mismatches, deletions, qualities) can be extracted against synthetic reference transcripts.

## When to use

Use when minimap2 has just produced raw SAM alignments of base-called ONT direct RNA reads (e.g., curlcake FASTQ vs. synthetic references) and sorted, per-condition BAM files are needed for downstream per-site feature extraction.

## When NOT to use

- Input is already a sorted BAM or feature table — re-converting adds no information and risks losing the mapping state
- Alignments come from a non-ONT platform or a different aligner preset, where '-ax map-ont' assumptions do not hold

## Inputs

- Raw SAM output from minimap2 '-ax map-ont' alignment of filtered base-called curlcake FASTQ reads
- Synthetic reference transcript sequences (four synthetic reference sequences)

## Outputs

- Coordinate-sorted BAM file per condition (m6A-modified IVT and unmodified IVT)
- Reads mapped to synthetic reference transcripts, ready for per-site error-feature extraction

## How to apply

After aligning NanoFilt-filtered FASTQ reads against the four synthetic reference transcripts with minimap2's '-ax map-ont' preset (chosen because the reads are Oxford Nanopore), pipe the raw SAM output into samtools to convert to BAM and coordinate-sort it (e.g., 'samtools sort -o condition.bam condition.sam'). Keep modified (m6A IVT) and unmodified (IVT) conditions as separate sorted BAMs so per-site comparison of base-called features between conditions is possible. Verify that one sorted BAM exists per condition and that reads are successfully mapped to the synthetic references before feeding them to EpiNano-style feature extraction (mismatch, insertion, deletion, per-base quality).

## Related tools

- **minimap2** (Aligns filtered ONT direct RNA reads to the four synthetic references using the '-ax map-ont' preset, producing the SAM that this skill converts)
- **samtools** (Converts minimap2 SAM output to BAM format and sorts it for downstream per-site error-feature extraction)
- **NanoFilt** (Upstream quality filtering and headcrop/tailcrop trimming of base-called FASTQ reads before alignment)
- **EpiNano** (Downstream consumer of the sorted BAMs — extracts per-kmer base-calling error features (mismatch, deletion, insertion, quality) for m6A prediction) — https://github.com/novoalab/EpiNano

## Examples

```
minimap2 -ax map-ont refs.fa filtered.fastq | samtools sort -o aligned.sorted.bam -
```

## Evaluation signals

- A sorted BAM file exists for each condition (m6A-modified and unmodified IVT)
- BAM files contain reads successfully mapped to the synthetic reference transcripts
- Read counts per BAM are consistent with the filtered FASTQ input (no silent loss during SAM→BAM conversion)
- Alignment used the '-ax map-ont' preset, appropriate for Oxford Nanopore reads
- Per-site feature extraction on the BAMs yields comparable coverage across modified and unmodified conditions

## Limitations

- Direct RNA base-calling is error-rich, so mapped reads alone do not imply modification detection — features must be compared between matched modified/unmodified samples
- EpiNano (the downstream consumer) has no per-read resolution and aggregates information across reads mapping to a site
- Pre-trained EpiNano-SVM models are base-caller dependent (Guppy 3.1.5 or Albacore 2.1.7); alignment/BAM handling does not mitigate base-caller version mismatch

## Evidence

- [methods] minimap2 map-ont: [evidence span withheld — anchor status: not_found]
- [methods] samtools sort BAM: [evidence span withheld — anchor status: not_found]
- [methods] validation check: [evidence span withheld — anchor status: not_found]
- [readme] README errors: [evidence span withheld — anchor status: external_doc]
