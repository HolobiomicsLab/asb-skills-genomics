---
name: sv-call-set-intersection-against-validated-junctions
description: Use when finished SV call sets (VCF) from multiple callers must be scored
  for recall against curated breakpoint-junction truth sets, e.g. validated de novo
  chromothripsis junctions per patient.
when_to_use_negative:
- Inputs are raw nanopore reads or unaligned FASTQ/BAM rather than finished SV call
  sets — run basecalling, LAST mapping, and NanoSV calling first
- No validated breakpoint-junction truth set exists for the sample, so recall cannot
  be computed (e.g. genome-wide inherited SV surveillance without curated de novo
  junctions)
- The goal is precision or false-discovery estimation rather than recovery of known
  junctions, which this recall-only intersection does not measure
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: NanoSV
  role: Caller under benchmark whose MinION-based VCF is intersected with the validated
    de novo junction sets (40/40 on Patient 1, 29/32 on Patient 2)
  repo: https://github.com/mroosmalen/nanosv
- name: Lumpy
  role: Comparison SV caller run on the same MinION nanopore BAMs; recall 33/40 (Patient
    1) and 9/32 (Patient 2)
- name: Sniffles
  role: Comparison long-read SV caller run on the same MinION nanopore BAMs; recall
    31/40 (Patient 1) and 16/32 (Patient 2)
- name: Manta
  role: Illumina short-read comparison caller evaluated on matched data; recall 19/32
    on Patient 2
- name: Delly
  role: Illumina short-read comparison caller evaluated on matched data; recall 22/32
    on Patient 2
- name: LAST
  role: Mapper that produces the sorted BAMs from which all benchmarked long-read
    caller outputs derive
  repo: http://last.cbrc.jp/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:fe67485b1519330371cc4858d81c3ad4f1766afa4a43590f49fd91b1b474b60e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - LAST
  - Lumpy
  - Manta
  - NanoSV
  - Sniffles
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# sv-call-set-intersection-against-validated-junctions

## Summary

Benchmarking step that intersects SV caller outputs (NanoSV, Lumpy, Sniffles on MinION nanopore data; Manta, Delly on matched Illumina data) with validated de novo chromothripsis breakpoint-junction truth sets to compute per-caller recall, quantifying the paper's claim that nanopore long reads outperform short reads for de novo rearrangement detection.

## When to use

Use when finished SV call sets (VCF) from multiple callers must be scored for recall against curated breakpoint-junction truth sets, e.g. validated de novo chromothripsis junctions per patient.

## When NOT to use

- Inputs are raw nanopore reads or unaligned FASTQ/BAM rather than finished SV call sets — run basecalling, LAST mapping, and NanoSV calling first
- No validated breakpoint-junction truth set exists for the sample, so recall cannot be computed (e.g. genome-wide inherited SV surveillance without curated de novo junctions)
- The goal is precision or false-discovery estimation rather than recovery of known junctions, which this recall-only intersection does not measure

## Inputs

- SV call sets in VCF format from NanoSV, Lumpy, and Sniffles run on LAST-mapped MinION nanopore BAMs
- SV call sets from Manta and Delly run on matched Illumina short-read BAMs
- Validated de novo breakpoint-junction truth sets (Patient 1: 40 junctions; Patient 2: 32 junctions)

## Outputs

- Per-caller, per-patient recovered-junction counts and recall fractions (e.g. NanoSV 40/40 on Patient 1, 29/32 on Patient 2)
- Recall benchmark table sv_caller_recall_benchmark.tsv

## How to apply

Intersect each caller's SV output with the validated de novo breakpoint-junction sets — 40 junctions for Patient 1 and 32 for Patient 2 from EGA study EGAS00001002333 — counting a validated junction as recovered when a caller reports a breakpoint on it. Compute recall per caller and per patient, keeping the caller-to-technology pairing intact: NanoSV, Lumpy, and Sniffles are evaluated on the same LAST-mapped MinION BAMs, while Manta and Delly are evaluated on matched Illumina short-read data. Write the recall table to sv_caller_recall_benchmark.tsv and verify it against the reported values (Patient 1: NanoSV 40/40, Lumpy 33/40, Sniffles 31/40; Patient 2: NanoSV 29/32, Lumpy 9/32, Sniffles 16/32, Manta 19/32, Delly 22/32). The rationale is to quantify the abstract's claim that nanopore long reads are superior to short reads for de novo chromothripsis detection: NanoSV recovers 100% of Patient 1 junctions even at relatively low coverage (16x), whereas the short-read callers recover substantially fewer.

## Related tools

- **NanoSV** (Caller under benchmark whose MinION-based VCF is intersected with the validated de novo junction sets (40/40 on Patient 1, 29/32 on Patient 2)) — https://github.com/mroosmalen/nanosv
- **Lumpy** (Comparison SV caller run on the same MinION nanopore BAMs; recall 33/40 (Patient 1) and 9/32 (Patient 2))
- **Sniffles** (Comparison long-read SV caller run on the same MinION nanopore BAMs; recall 31/40 (Patient 1) and 16/32 (Patient 2))
- **Manta** (Illumina short-read comparison caller evaluated on matched data; recall 19/32 on Patient 2)
- **Delly** (Illumina short-read comparison caller evaluated on matched data; recall 22/32 on Patient 2)
- **LAST** (Mapper that produces the sorted BAMs from which all benchmarked long-read caller outputs derive) — http://last.cbrc.jp/

## Evaluation signals

- sv_caller_recall_benchmark.tsv reproduces the reported counts exactly: Patient 1 NanoSV 40/40, Lumpy 33/40, Sniffles 31/40
- Patient 2 values match the paper: NanoSV 29/32, Lumpy 9/32, Sniffles 16/32, Manta 19/32, Delly 22/32
- Recall counts never exceed the truth-set denominators (40 junctions for Patient 1, 32 junctions for Patient 2)
- NanoSV attains 100% recall on Patient 1 despite only 11-16x nanopore coverage — any missed validated junction signals a mapping or intersection bug
- Caller-to-technology pairing is preserved: NanoSV/Lumpy/Sniffles scored on MinION BAMs, Manta/Delly on matched Illumina data

## Limitations

- Recall estimates are limited by the medium coverage (11-16x) and intrinsic nanopore sequencing biases of the MinION data
- Truth sets cover only validated de novo chromothripsis junctions (40 and 32), so this benchmark says nothing about precision or novel inherited SVs, where the pipeline's extremely stringent filtering yields an estimated sensitivity of ~75%
- Even NanoSV does not recover all junctions on Patient 2 nanopore data (29/32), so 100% recovery cannot be assumed for every patient

## Evidence

- [abstract] nanopore superiority for de novo detection: "nanopore long reads are superior to short reads with regard to detection of de novo chromothripsis rearrangements"
- [discussion] full recovery of Patient 1 junctions at low coverage: "We were able to extract all known de novo breakpoint junctions for Patient 1 (Fig. 1), even at relatively low coverage (16×)"
- [discussion] stringent filtering lowers estimated sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] coverage and bias caveats: "Due to the medium coverage, some intrinsic nanopore sequencing biases and for benchmarking purposes"
- [readme] NanoSV breakpoint-junction algorithm: "identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
