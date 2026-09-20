---
name: vcf-variant-callset-handling
description: Use when handling TE insertion callsets from nanopore-sequenced samples
  — e.g. polished bgzipped VCFs from cuteSV/Sniffles2 that need supporting-read filtering,
  0/0 genotype removal, or SVA F1 re-labeling before downstream annotation or sample
  comparison.
when_to_use_negative:
- Input is short-read data analyzed with MELT-style tools rather than nanopore data
  — RetroInspector's callset handling is designed for nanopore-sequenced human genomes.
- Input is raw basecalled FASTQ/FASTA rather than a variant callset — run the RetroInspector
  detection and reassembly steps first (the pipeline requires the hg38/GRCh38 reference
  genome).
- Input is a non-human genome or non-GRCh38 reference — the pipeline is built around
  the human reference and TE families covered by Dfam.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3263
tools:
- name: RetroInspector
  role: Snakemake pipeline that produces, filters, labels (SVA F1), and writes the
    bgzipped VCF callsets
  repo: github.com/javiercguard/retroinspector
- name: cuteSV
  role: variant caller whose insertion records feed the polished VCF callset
  repo: github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: second variant caller whose records are combined with cuteSV in the callset
  repo: github.com/fritzsedlazeck/Sniffles
- name: BLAST
  role: queries SVA insertion sequences against the MAST2 fragment to label records
    as SVA F1
- name: pysam
  role: library used for reading and writing VCF callset records in the pipeline
  repo: github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:bc2f3112c3a000a6c9f7d3f427ff9b5508f1d5ca6f0328846f38452c93682b7f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  tools:
  - BLAST
  - RetroInspector
  - Sniffles2
  - cuteSV
  - pysam
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3263
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# vcf-variant-callset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load, filter, re-label, and write structural-variant callsets as bgzipped VCFs in the RetroInspector nanopore TE pipeline: applying supporting-read thresholds, dropping homozygous-reference genotypes, and adding post-hoc classifications such as SVA F1 identification via the MAST2 thumbprint.

## When to use

Use when handling TE insertion callsets from nanopore-sequenced samples — e.g. polished bgzipped VCFs from cuteSV/Sniffles2 that need supporting-read filtering, 0/0 genotype removal, or SVA F1 re-labeling before downstream annotation or sample comparison.

## When NOT to use

- Input is short-read data analyzed with MELT-style tools rather than nanopore data — RetroInspector's callset handling is designed for nanopore-sequenced human genomes.
- Input is raw basecalled FASTQ/FASTA rather than a variant callset — run the RetroInspector detection and reassembly steps first (the pipeline requires the hg38/GRCh38 reference genome).
- Input is a non-human genome or non-GRCh38 reference — the pipeline is built around the human reference and TE families covered by Dfam.

## Inputs

- bgzipped polished VCF of TE insertion/deletion calls (e.g. S1.cutesv.polished.vcf.gz) from cuteSV/Sniffles2 consensus reassembly
- supporting-read counts and genotypes per variant record
- reassembled insertion sequences per sample
- MAST2 fragment reference sequence for SVA F1 thumbprint search

## Outputs

- filtered and labelled bgzipped VCF callset with SVA F1 annotations added to insertion records
- retained records carrying ≥3 supporting reads and no 0/0 genotypes
- per-record SVA F1 flag backed by a BLAST hit to the MAST2 fragment

## How to apply

Load the reassembled inserted sequences from the polished VCFs produced by the consensus reassembly step (e.g. S1.cutesv.polished.vcf.gz), keeping records called by cuteSV and Sniffles2. Filter the callset with the pipeline's default (and recommended) criterion of at least 3 supporting reads, choosing between the strict and lax filtering criteria evaluated in the article, and drop variants genotyped 0/0 (homozygous for the reference allele) across samples. As a discrete post-hoc classification step, query called SVA insertion sequences with BLAST against the MAST2 fragment carried by SVA F1 elements (SVA F1 is absent from Dfam, so the RepeatMasker-on-Dfam step reports these only as generic SVA), and label matching records SVA F1. Write the labelled callset to a bgzipped VCF consistent with the other RetroInspector variant outputs so it remains indexable and interoperable.

## Related tools

- **RetroInspector** (Snakemake pipeline that produces, filters, labels (SVA F1), and writes the bgzipped VCF callsets) — github.com/javiercguard/retroinspector
- **cuteSV** (variant caller whose insertion records feed the polished VCF callset) — github.com/tjiangHIT/cuteSV
- **Sniffles2** (second variant caller whose records are combined with cuteSV in the callset) — github.com/fritzsedlazeck/Sniffles
- **BLAST** (queries SVA insertion sequences against the MAST2 fragment to label records as SVA F1)
- **pysam** (library used for reading and writing VCF callset records in the pipeline) — github.com/pysam-developers/pysam

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Output VCF exists and parses as a valid VCF.
- Every record flagged SVA F1 carries a BLAST hit to the MAST2 fragment sequence (confirm by expert review of a sample of hits).
- All retained records show at least 3 supporting reads (the pipeline's default and recommended setting).
- No records remain with a 0/0 (homozygous reference) genotype across samples.
- Output is bgzipped, consistent with other RetroInspector variant outputs (indexable with tabix).

## Limitations

- SVA F1 is not included in Dfam, so the RepeatMasker-on-Dfam step reports these insertions only as generic SVA; classification depends entirely on the discrete MAST2 thumbprint check.
- Publicly available HGSVC data was basecalled with an older basecaller, so even after reassembly, sequences may be imperfect, particularly for homopolymers.
- Detailed callset/filter settings are only documented in the pipeline repository's template configuration file, not in the article text.

## Evidence
