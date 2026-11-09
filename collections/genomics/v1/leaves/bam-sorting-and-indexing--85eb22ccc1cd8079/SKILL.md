---
name: bam-sorting-and-indexing
description: Use when basecalled DRS reads have been aligned to a reference transcriptome
  (e.g. with minimap2) and downstream steps need a sorted, indexed BAM — such as samtools
  view filtering, jvarkit sam2tsv conversion, or feature extraction in RedNano.
when_to_use_negative:
- Alignments already coordinate-sorted and indexed (re-sorting is redundant)
- Alignment to a whole genome reference instead of a transcriptome — RedNano's resquiggle
  and feature extraction assume transcript-level coordinates
- When only basecalled FASTQ exists without an alignment step having been run first
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Produces the initial SAM alignment (-t 30 -ax map-ont) of Nanopore reads to
    the reference transcriptome, which is piped into the sort step
  repo: https://github.com/Derryxu/RedNano
- name: samtools
  role: Sorts the aligned BAM (-@ 30) and indexes it (samtools index); v1.7 used in
    RedNano
  repo: https://github.com/Derryxu/RedNano
- name: jvarkit sam2tsv
  role: Consumes the sorted/indexed BAM (after samtools view -F 3844) to emit per-read
    TSV used for feature extraction
  repo: https://github.com/lindenb/jvarkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:58165b15b4c71d8f569b6a87de797c7d7e9edd4f326dea7d95233d4434452991
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - RedNano
  - jvarkit sam2tsv
  - minimap2
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bam-sorting-and-indexing

## Summary

Sort position-sorted BAM alignments of Nanopore DRS reads to a reference transcriptome and index them with samtools, as required in the RedNano preprocessing pipeline before read-feature extraction and m6A site prediction.

## When to use

Use when basecalled DRS reads have been aligned to a reference transcriptome (e.g. with minimap2) and downstream steps need a sorted, indexed BAM — such as samtools view filtering, jvarkit sam2tsv conversion, or feature extraction in RedNano.

## When NOT to use

- Alignments already coordinate-sorted and indexed (re-sorting is redundant)
- Alignment to a whole genome reference instead of a transcriptome — RedNano's resquiggle and feature extraction assume transcript-level coordinates
- When only basecalled FASTQ exists without an alignment step having been run first

## Inputs

- SAM/BAM stream from minimap2 (-ax map-ont) alignment of basecalled reads (test.fastq) to a reference transcriptome FASTA (reference.transcript.fa)
- samtools v1.7 executable

## Outputs

- Coordinate-sorted BAM file (test.bam)
- BAM index file (test.bam.bai)
- Downstream TSV of per-read alignment records (test.tsv) after filtering (-F 3844) and jvarkit sam2tsv conversion

## How to apply

In the RedNano pipeline, minimap2 (v2.17) aligns the merged basecalled FASTQ (test.fastq) to the reference transcriptome with `-t 30 -ax map-ont`; the output is piped through `samtools view -hSb` to emit clean BAM records and then `samtools sort -@ 30 -o test.bam` to produce a coordinate-sorted BAM. Run `samtools index test.bam` to create the .bai index, which is required before filtered views or the sam2tsv (jvarkit) conversion step. Sorted BAM was generated with samtools v1.7 in this workflow. The sort-and-index step is what enables deterministic, parallelizable downstream extraction: the indexed BAM is filtered (`samtools view -h -F 3844`) to drop unmapped/secondary/supplementary records and converted to a per-read TSV for feature extraction.

## Related tools

- **minimap2** (Produces the initial SAM alignment (-t 30 -ax map-ont) of Nanopore reads to the reference transcriptome, which is piped into the sort step) — https://github.com/Derryxu/RedNano
- **samtools** (Sorts the aligned BAM (-@ 30) and indexes it (samtools index); v1.7 used in RedNano) — https://github.com/Derryxu/RedNano
- **jvarkit sam2tsv** (Consumes the sorted/indexed BAM (after samtools view -F 3844) to emit per-read TSV used for feature extraction) — https://github.com/lindenb/jvarkit

## Examples

```
minimap2 -t 30 -ax map-ont referance.transcript.fa test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam; samtools index test.bam
```

## Evaluation signals

- test.bam exists and samtools quickcheck passes (valid BAM container)
- BAM is coordinate-sorted (SO:coordinate header flag set by samtools sort)
- test.bam.bai index file generated by samtools index is present
- samtools view -F 3844 on the indexed BAM returns mapped primary alignments only (3844 = unmapped + secondary + supplementary)
- sam2tsv output (test.tsv) contains per-read alignment columns consistent with the reference transcriptome IDs

## Limitations

- RedNano's workflow requires alignment against the reference transcriptome (reference gene fasta), not a genome file — sorting a genome-aligned BAM breaks downstream steps
- Reads with flags in 3844 (unmapped/secondary/supplementary) are discarded before feature extraction, so sorting/indexing alone is insufficient without the filtering step

## Evidence

- [readme] minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam: "minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam"
- [readme] samtools index test.bam: "samtools index test.bam"
- [readme] samtools view -h -F 3844 test.bam | java -jar sam2tsv.jar -r referance.transcript.fa > test.tsv: "samtools view -h -F 3844 test.bam | java -jar sam2tsv.jar -r referance.transcript.fa > test.tsv"
- [methods] each basecalled read is aligned to the reference transcriptome using minimap2 (v2.17), and raw current signals are mapped to contiguous reference bases: [evidence span withheld — anchor status: not_found]
