---
name: samtools-read-manipulation
description: Use when input is a sorted/aligned BAM (e.g. SARS-CoV-2 amplicon reads)
  and you need to subsample to defined mapped-read depths (100–1M), index/sort alignments,
  or pipe mapper output into downstream consensus calling.
when_to_use_negative:
- Input is raw FASTQ reads not yet aligned — align first (e.g. with minimap2 '-a -x
  sr' against the reference) before samtools manipulation
- Reference genome is ultra-long (e.g. human chromosomes) — memory of downstream consensus
  callers scales linearly with reference genome length
- Goal is variant calling or full pipeline reconstruction (e.g. CoVpipe2) rather than
  read-level BAM manipulation or consensus-calling benchmarking
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0769
tools:
- name: samtools
  role: Sorting, indexing, filtering, and subsampling BAM alignments; converting mapper
    output to BAM; counting mapped reads per subsample
  repo: https://github.com/samtools/htslib
- name: htslib
  role: Core C library for SAM/BAM access underlying samtools and ViralConsensus;
    must be installed and on LD_LIBRARY_PATH
  repo: https://github.com/samtools/htslib
- name: ViralConsensus
  role: Downstream consensus caller benchmarked on samtools-subsampled BAMs; consumes
    sorted BAM via '-i'
  repo: https://github.com/niemasd/ViralConsensus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:7167c2fb9c4967e5546df34e0ba7aa918778b501694083e6fd070088e44caedf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samtools/htslib
  tools:
  - ViralConsensus
  - htslib
  - samtools
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# samtools-read-manipulation

## Summary

Use samtools (htslib) to manipulate sorted/aligned BAM read data — indexing, sorting, filtering, and read subsampling by mapped-read count — as the substrate for benchmarking consensus callers such as ViralConsensus and the iVar pipeline on SARS-CoV-2 amplicon sequencing data.

## When to use

Use when input is a sorted/aligned BAM (e.g. SARS-CoV-2 amplicon reads) and you need to subsample to defined mapped-read depths (100–1M), index/sort alignments, or pipe mapper output into downstream consensus calling.

## When NOT to use

- Input is raw FASTQ reads not yet aligned — align first (e.g. with minimap2 '-a -x sr' against the reference) before samtools manipulation
- Reference genome is ultra-long (e.g. human chromosomes) — memory of downstream consensus callers scales linearly with reference genome length
- Goal is variant calling or full pipeline reconstruction (e.g. CoVpipe2) rather than read-level BAM manipulation or consensus-calling benchmarking

## Inputs

- Sorted/aligned BAM file of sequencing reads (e.g. SARS-CoV-2 amplicon sequencing sample)
- Reference FASTA (e.g. NC_045512.2.fas) used by downstream consensus callers
- Subsampled BAM files at target depths (100, 1K, 10K, 100K, 1M mapped reads)

## Outputs

- Indexed, sorted subsampled BAM files per replicate and depth
- Mapped-read counts per subsample for aggregation into benchmark_results.csv
- BAM stream piped via standard input to consensus callers

## How to apply

Start from a sorted, aligned BAM of the highest-depth sample and use samtools view-based read subsampling to create replicates at five depths: 100, 1K, 10K, 100K, and 1M mapped reads (10 replicates per depth for benchmarking). Index subsampled BAMs with samtools when the downstream tool requires indexed input (as the iVar consensus pipeline does). samtools also converts raw mapper output to BAM ('samtools view -b -@ 1 > reads.bam') and its underpinning, htslib, is the library ViralConsensus itself depends on. Record the actual mapped-read count per subsample (via samtools flagstat or view -c) so runtime and peak-RSS measurements can be plotted against depth when aggregating results into benchmark_results.csv.

## Related tools

- **samtools** (Sorting, indexing, filtering, and subsampling BAM alignments; converting mapper output to BAM; counting mapped reads per subsample) — https://github.com/samtools/htslib
- **htslib** (Core C library for SAM/BAM access underlying samtools and ViralConsensus; must be installed and on LD_LIBRARY_PATH) — https://github.com/samtools/htslib
- **ViralConsensus** (Downstream consensus caller benchmarked on samtools-subsampled BAMs; consumes sorted BAM via '-i') — https://github.com/niemasd/ViralConsensus

## Examples

```
samtools view -b -s 0.01 -@ 1 high_depth.sorted.bam > subsample_10K.bam && samtools index subsample_10K.bam
```

## Evaluation signals

- Each subsample's mapped-read count (samtools view -c) matches its target depth (100, 1K, 10K, 100K, 1M) within tolerance
- BAM files are coordinate-sorted and indexed (samtools quickcheck passes; .bai/.csi index present) before downstream consensus calling
- Subsampling is reproducible across the 10 replicates per depth (same seed/strategy documented)
- Downstream benchmark confirms ViralConsensus runtime scales linearly with mapped-read count while peak memory stays constant (~under 12 MB at 1M reads)

## Limitations

- Read subsampling preserves the original alignment's properties; it does not simulate independent sequencing replicates
- samtools-based subsampling requires coordinate-sorted, aligned input with sufficient depth — the highest-depth sample bounds the maximum achievable subsample depth
- htslib must be installed in the library path (LD_LIBRARY_PATH=/usr/local/lib) or dependent tools fail to load

## Evidence

- [other] Subsample the sorted/aligned reads of the highest-depth SARS-CoV-2 sample at five depths (100, 1K, 10K, 100K, 1M mapped reads) with 10 replicates per depth, using samtools-based read subsampling.: [evidence span withheld — anchor status: not_found]
- [methods] samtools converts mapper output to BAM.: "samtools view -b -@ 1 > reads.bam"
- [methods] ViralConsensus depends on htslib.: "ViralConsensus is written in C++ and depends on htslib"
- [readme] README describes htslib BAM indexing.: "HTSlib implements a generalized BAM index, with file extension `.csi`
(coordinate-sorted index)."
