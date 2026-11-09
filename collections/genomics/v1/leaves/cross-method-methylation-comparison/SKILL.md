---
name: cross-method-methylation-comparison
description: Use when validating a new methylation-calling method against established
  platforms (WGBS, EMseq, nanopolish) by comparing overall CpG methylation rates and
  CpG coverage per sample.
when_to_use_negative:
- The goal is to distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the
  same DNA molecule — nanoEM cannot separate mC from hmC, unlike direct nanopore WGS
  calling
- Input reads are short-read bisulfite/enzymatic data already processed by standard
  aligners — this skill targets long-read nanoEM data and its CT/GA conversion pipeline,
  not short-read WGBS/EMseq processing
- Direct single-molecule methylation calling from native DNA with sufficient input
  (nanopore WGS requires ~500 ng / 150 fmol) is feasible — then nanopolish direct
  calling, not the comparison-by-reconversion route, is the simpler choice
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: nanopore sequencing
  role: Long-read sequencing platform whose output feeds both the nanoEM and nanopolish
    methylation calls being compared
- name: minimap2
  role: Aligns CT- and GA-converted nanoEM reads to the converted reference (-t 8
    --split-prefix -ax map-ont --eqx) before methylation calling
- name: samtools
  role: Sorts and indexes the BAM/SAM outputs of minimap2 and best_align.py prior
    to mpileup
- name: sambamba
  role: Runs mpileup on both sorted BAMs restricted to cpg_sites.bed to produce pileup_CT.tsv
    and pileup_GA.tsv
- name: nanopolish
  role: Calls methylation directly from native PromethION WGS reads, providing the
    direct-calling comparison rates (55.5% MB231, 79.8% BT474)
  repo: jts/nanopolish
- name: Python3
  role: Runs the nanoEM pipeline scripts (convert_ref.py, convert_reads.py, best_align.py,
    call_methylation.py)
  repo: https://github.com/yos-sk/nanoEM
- name: pysam
  role: Python dependency of the nanoEM pipeline scripts used for BAM/SAM manipulation
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:65a6a532b836df950a70e44477620be77e071e871c22cac718f275cc16a816d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/jts/nanopolish
  tools:
  - Python3
  - minimap2
  - nanopolish
  - nanopore sequencing
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-method-methylation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces and compares genome-wide CpG methylation rates measured by nanoEM (long-read enzymatic conversion + nanopore) against short-read WGBS, EMseq, and nanopolish direct nanopore calling, in the breast cancer cell lines MB231 and BT474.

## When to use

Use when validating a new methylation-calling method against established platforms (WGBS, EMseq, nanopolish) by comparing overall CpG methylation rates and CpG coverage per sample.

## When NOT to use

- The goal is to distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same DNA molecule — nanoEM cannot separate mC from hmC, unlike direct nanopore WGS calling
- Input reads are short-read bisulfite/enzymatic data already processed by standard aligners — this skill targets long-read nanoEM data and its CT/GA conversion pipeline, not short-read WGBS/EMseq processing
- Direct single-molecule methylation calling from native DNA with sufficient input (nanopore WGS requires ~500 ng / 150 fmol) is feasible — then nanopolish direct calling, not the comparison-by-reconversion route, is the simpler choice

## Inputs

- NanoEM nanopore FASTQ reads (DRA011237, MB231 and BT474)
- PromethION WGS nanopore reads for nanopolish methylation calling
- Nanopolish methylation call files (E-GEAD-408)
- Deposited short-read WGBS and EMseq methylation calls
- Reference genome FASTA and cpg_sites.bed CpG coordinates

## Outputs

- methylation_comparison_table.tsv with a methylation-rate column and a covered-CpG-count column for nanoEM, WGBS, EMseq, and nanopolish
- Per-CpG methylation frequencies (frequency_methylation.tsv) for each sample
- Reproduced values for Figures 1A-1C and Supplementary Tables S1, S2, S5

## How to apply

Download nanopore reads from DRA011237 and nanopolish methylation calls from E-GEAD-408, then run the nanoEM pipeline: convert the reference genome (convert_ref.py) and nanoEM reads into CT and GA fastq files (convert_reads.py), align each with minimap2 (-t 8 --split-prefix -ax map-ont --eqx), sort and index with samtools, select best alignments across the two strand conversions with best_align.py, and call per-CpG methylation by running sambamba mpileup on both sorted BAMs restricted to cpg_sites.bed and combining pileups with call_methylation.py. Compute CpG coverage at >=5 reads, extract overall CpG methylation rates from the WGBS and nanopolish (PromethION WGS) deposited calls, and combine all per-method rates and covered-CpG counts into methylation_comparison_table.tsv. Validate by checking nanoEM rates (45.7% MB231, 79.2% BT474) against WGBS (53.3%/55.1% and 82.5%/84.9%) and nanopolish (55.5%/79.8%) within reasonable tolerance.

## Related tools

- **nanopore sequencing** (Long-read sequencing platform whose output feeds both the nanoEM and nanopolish methylation calls being compared)
- **minimap2** (Aligns CT- and GA-converted nanoEM reads to the converted reference (-t 8 --split-prefix -ax map-ont --eqx) before methylation calling)
- **samtools** (Sorts and indexes the BAM/SAM outputs of minimap2 and best_align.py prior to mpileup)
- **sambamba** (Runs mpileup on both sorted BAMs restricted to cpg_sites.bed to produce pileup_CT.tsv and pileup_GA.tsv)
- **nanopolish** (Calls methylation directly from native PromethION WGS reads, providing the direct-calling comparison rates (55.5% MB231, 79.8% BT474)) — jts/nanopolish
- **Python3** (Runs the nanoEM pipeline scripts (convert_ref.py, convert_reads.py, best_align.py, call_methylation.py)) — https://github.com/yos-sk/nanoEM
- **pysam** (Python dependency of the nanoEM pipeline scripts used for BAM/SAM manipulation) — https://github.com/yos-sk/nanoEM

## Examples

```
python src/call_methylation.py pileup_CT.tsv pileup_GA.tsv > frequency_methylation.tsv
```

## Evaluation signals

- nanoEM overall CpG methylation rates reproduce the reported values: 45.7% (MB231) and 79.2% (BT474)
- WGBS rates fall at 53.3%/55.1% and 82.5%/84.9%, and nanopolish rates at 55.5%/79.8%, within reasonable tolerance
- methylation_comparison_table.tsv contains both a methylation-rate column and a covered-CpG-count column for each of the three methods
- CpG coverage is computed with the >=5-read threshold and covered-CpG counts are populated for all methods
- Reproduced values match Figures 1A-1C and Supplementary Tables S1, S2, S5

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, whereas direct nanopore WGS calling has that potential
- nanoEM's MB231 methylation rate (45.7%) was somewhat lower than the short-read WGBS and EMseq values
- No software version numbers are stated for minimap2, nanopolish, or the pipeline scripts, complicating exact reproduction
- Direct nanopore WGS methylation calling requires ~500 ng (150 fmol) DNA input, so the comparison route via nanoEM exists because nanoEM works from as little as 1 ng genomic DNA

## Evidence

- [results] nanoEM's overall CpG methylation rates (45.7% MB231, 79.2% BT474) were almost consistent with short-read WGBS and EMseq, though nanoEM in MB231 was somewhat lower.: [evidence span withheld — anchor status: not_found]
