---
name: illumina-short-read-sv-calling
description: Use when matched Illumina short-read BAM/CRAM data are available for
  a patient genome and you need SV calls (or a short-read recall baseline) to compare
  against long-read callers on validated breakpoint-junction sets.
when_to_use_negative:
- Input is nanopore long-read data — NanoSV with LAST mapping is the appropriate caller
  pipeline for that modality
- Only a small targeted region is of interest rather than genome-wide SV discovery
  on whole-genome short-read data
- No validated breakpoint-junction truth set exists and the goal is de novo discovery
  rather than caller recall benchmarking
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Manta
  role: 'Short-read SV caller run on matched Illumina data to compute recall of validated
    de novo breakpoints (Patient 2: 19/32)'
- name: Delly
  role: 'Short-read SV caller run on matched Illumina data to compute recall of validated
    de novo breakpoints (Patient 2: 22/32)'
- name: NanoSV
  role: Long-read reference caller whose recall (e.g., 29/32 on Patient 2 nanopore
    data) is the comparison baseline for the short-read callers
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Used to sort and process BAM alignments feeding the SV calling pipeline
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:0f4b2162f7e19d8e4a06a31f256740c558800189dbad0531925c5aee891da147
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - Illumina
  - Lumpy
  - Manta
  - NanoSV
  - Sniffles
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# illumina-short-read-sv-calling

## Summary

Calls structural variants from matched Illumina short-read data (e.g., with Manta and Delly) to benchmark their recall against long-read nanopore SV callers such as NanoSV on the same patient genomes. In the source study this established that short-read callers miss de novo chromothripsis breakpoints that NanoSV recovers from MinION reads.

## When to use

Use when matched Illumina short-read BAM/CRAM data are available for a patient genome and you need SV calls (or a short-read recall baseline) to compare against long-read callers on validated breakpoint-junction sets.

## When NOT to use

- Input is nanopore long-read data — NanoSV with LAST mapping is the appropriate caller pipeline for that modality
- Only a small targeted region is of interest rather than genome-wide SV discovery on whole-genome short-read data
- No validated breakpoint-junction truth set exists and the goal is de novo discovery rather than caller recall benchmarking

## Inputs

- Coordinate-sorted, indexed BAM files from matched Illumina short-read sequencing of the patient genomes (EGA study EGAS00001002333)
- Validated de novo breakpoint-junction sets (Patient 1: 40 junctions; Patient 2: 32 junctions) for recall computation
- Human reference genome used for alignment and BAM preparation

## Outputs

- SV VCF files per short-read caller (Manta, Delly) per patient
- Per-caller recall table intersected with validated breakpoint junctions (sv_caller_recall_benchmark.tsv, e.g., Patient 2: Manta 19/32, Delly 22/32)

## How to apply

Run short-read SV callers such as Manta and Delly on the coordinate-sorted, indexed BAM files derived from the matched Illumina sequencing data for each patient (here, Patient 2 from EGA study EGAS00001002333). Intersect each caller's VCF output with the validated de novo breakpoint-junction set (e.g., 32 junctions for Patient 2, 40 for Patient 1) and compute per-caller recall. In the source study, short-read callers recovered only a fraction of validated junctions (Patient 2: Manta 19/32, Delly 22/32) compared to NanoSV on nanopore reads (29/32), reflecting the abstract's conclusion that nanopore long reads are superior to short reads for detecting de novo chromothripsis rearrangements. Use the resulting recall table (e.g., sv_caller_recall_benchmark.tsv) as the correctness check against reported values.

## Related tools

- **Manta** (Short-read SV caller run on matched Illumina data to compute recall of validated de novo breakpoints (Patient 2: 19/32))
- **Delly** (Short-read SV caller run on matched Illumina data to compute recall of validated de novo breakpoints (Patient 2: 22/32))
- **NanoSV** (Long-read reference caller whose recall (e.g., 29/32 on Patient 2 nanopore data) is the comparison baseline for the short-read callers) — https://github.com/mroosmalen/nanosv
- **sambamba** (Used to sort and process BAM alignments feeding the SV calling pipeline) — https://github.com/biod/sambamba

## Examples

```
manta/bin/configManta.py --bam patient2.illumina.sorted.bam --referenceFasta hg19.fa --runDir manta_p2 && manta_p2/runWorkflow.py -m local -j 8; delly call -g hg19.fa patient2.illumina.sorted.bam > delly_p2.bcf
```

## Evaluation signals

- Each caller's VCF is intersected with the validated junction set and recall matches reported values (Patient 2: Manta 19/32, Delly 22/32; Patient 1: NanoSV 40/40, Lumpy 33/40, Sniffles 31/40)
- sv_caller_recall_benchmark.tsv is written with one row per caller-patient and matches the reported counts exactly
- Input BAMs are coordinate-sorted and indexed before SV calling
- Per-patient breakdown is preserved (junction totals differ: 40 for Patient 1, 32 for Patient 2)

## Limitations

- Short-read callers recover only a minority of validated de novo chromothripsis breakpoints (Manta 19/32, Delly 22/32 on Patient 2), so they are not a substitute for long-read SV calling on complex rearrangements
- The paper notes variants 'could not be found in matching Illumina sequencing data (despite extensive variant calling using six different variant calling methods)'
- Stringent filtering (as in the nanopore pipeline) trades precision for sensitivity; estimated sensitivity of the study's SV pipeline was ~75%

## Evidence

- [abstract] Short-read callers underperform on de novo breakpoints: "nanopore long reads are superior to short reads with regard to detection of de novo chromothripsis rearrangements"
- [discussion] Variants missed in matching Illumina data: "could not be found in matching Illumina sequencing data (despite extensive variant calling using six different variant calling methods"
- [discussion] Study recall benchmark with six variant calling methods: "SV calls matching any state of the art precision benchmark (>95% precision)"
- [readme] NanoSV requires sorted, indexed long-read BAMs (contrast to short-read workflow): "This BAM file needs to be coordinate-sorted and indexed."
