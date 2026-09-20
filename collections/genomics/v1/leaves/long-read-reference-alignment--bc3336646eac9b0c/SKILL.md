---
name: long-read-reference-alignment
description: Use when raw per-sample long-read FASTQ files (PacBio HiFi or targeted
  ONT) must be aligned to a human reference genome FASTA to yield coordinate-sorted,
  indexed BAM/CRAM files as the basis for phasing, variant calling, or assembly.
when_to_use_negative:
- Input is already an aligned (u)BAM that only needs sorting/indexing rather than
  read-to-reference alignment
- The analysis goal is de novo genome assembly from reads rather than mapping to a
  reference (use Hifiasm instead, though assembled haplotypes are later aligned with
  Minimap2)
- Short-read sequencing data is the input, since Minimap2 long-read presets and the
  Nallo long-read workflow are designed for PacBio/ONT data
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Minimap2
  role: Maps the long reads to the reference genome using chemistry-appropriate presets
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts, merges and indexes the aligned reads to produce coordinate-sorted BAM/CRAM
    with index
  repo: https://github.com/samtools/samtools
- name: Nextflow
  role: Workflow engine in which the alignment process is implemented as part of the
    Nallo pipeline
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: nf-test
  role: Runs the module's test on bundled test data to confirm the alignment outputs
    match expected snapshots
  repo: https://github.com/genomic-medicine-sweden/nallo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:2313c3ae562a9c52c1a3bf410c1d604c854493752073e2dd597d4276a49636e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Docker
  - Minimap2
  - Nallo
  - Nextflow
  - nf-test
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-reference-alignment

## Summary

Maps long-read sequencing data (PacBio HiFi or ONT) to a human reference genome with Minimap2, then sorts, merges and indexes the aligned reads with samtools to produce per-sample coordinate-sorted BAM/CRAM files with .bai/.crai indexes for downstream phasing, variant calling and assembly.

## When to use

Use when raw per-sample long-read FASTQ files (PacBio HiFi or targeted ONT) must be aligned to a human reference genome FASTA to yield coordinate-sorted, indexed BAM/CRAM files as the basis for phasing, variant calling, or assembly.

## When NOT to use

- Input is already an aligned (u)BAM that only needs sorting/indexing rather than read-to-reference alignment
- The analysis goal is de novo genome assembly from reads rather than mapping to a reference (use Hifiasm instead, though assembled haplotypes are later aligned with Minimap2)
- Short-read sequencing data is the input, since Minimap2 long-read presets and the Nallo long-read workflow are designed for PacBio/ONT data

## Inputs

- Per-sample long-read FASTQ files (PacBio HiFi or ONT)
- Human reference genome FASTA

## Outputs

- Per-sample coordinate-sorted BAM or CRAM alignment file
- Alignment index file (.bai or .crai)

## How to apply

Load per-sample long-read FASTQ files and the human reference genome FASTA as channel inputs to the alignment process. Align reads to the reference with Minimap2, selecting the preset appropriate to the input chemistry (e.g. map-hifi for PacBio HiFi, lr:hq for high-quality ONT) since preset choice strongly affects mapping sensitivity. Sort, merge, and index the aligned reads with samtools (samtools sort, samtools merge, samtools index), producing per-sample coordinate-sorted BAM/CRAM plus a .bai/.crai index. Publish the sorted alignment file and index to the results directory and emit them as process outputs for downstream phasing, variant calling, and assembly steps; if phasing is enabled, the aligned reads will subsequently be haplotagged by the active phasing tool.

## Related tools

- **Minimap2** (Maps the long reads to the reference genome using chemistry-appropriate presets) — https://github.com/lh3/minimap2
- **samtools** (Sorts, merges and indexes the aligned reads to produce coordinate-sorted BAM/CRAM with index) — https://github.com/samtools/samtools
- **Nextflow** (Workflow engine in which the alignment process is implemented as part of the Nallo pipeline) — https://github.com/genomic-medicine-sweden/nallo
- **nf-test** (Runs the module's test on bundled test data to confirm the alignment outputs match expected snapshots) — https://github.com/genomic-medicine-sweden/nallo

## Examples

```
nextflow run genomic-medicine-sweden/nallo --input samplesheet.csv --genome GRCh38 -profile docker
```

## Evaluation signals

- The pipeline's bundled nf-test for the alignment module passes when sorted, merged and indexed BAM/CRAM plus index files are produced
- Module snapshot outputs match the expected BAM/CRAM and .bai/.crai files on bundled test data
- The alignment file is coordinate-sorted and accompanied by a valid .bai/.crai index
- The published results directory contains the per-sample alignment file and index as process outputs for downstream steps

## Limitations

- Minimap2 preset must match the input read chemistry (map-hifi for PacBio HiFi, lr:hq for high-quality ONT); a mismatched preset degrades mapping
- When the pipeline runs with phasing, aligned reads are haplotagged by the active phasing tool, so alignment outputs may be modified downstream
- Nallo is focused on rare-disease long-read data from PacBio and (targeted) ONT and is not a generic alignment workflow for arbitrary data types

## Evidence

- [intro] Minimap2 maps long reads to human reference; samtools sorts, merges and indexes aligned reads: "[Minimap2](https://github.com/lh3/minimap2) is used to map the reads to a reference genome."
- [intro] Aligned reads are sorted, merged and indexed with samtools: "The aligned reads are sorted, merged and indexed using [samtools](https://github.com/samtools/samtools)."
- [abstract] Nallo is a Nextflow pipeline for PacBio and Oxford Nanopore data: "Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data"
- [intro] Aligned reads are haplotagged when phasing is enabled: "If the pipeline is run with phasing, the aligned reads will be haplotagged using the active phasing tool."
