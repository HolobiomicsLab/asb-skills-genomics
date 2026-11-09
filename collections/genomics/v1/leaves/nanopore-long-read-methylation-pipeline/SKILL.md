---
name: nanopore-long-read-methylation-pipeline
description: Use when analyzing EMseq/nanoEM base-converted Oxford Nanopore long reads
  to quantify per-CpG methylation genome-wide, or when reproducing methylation rates
  and CpG coverage versus WGBS, EMseq, and nanopolish.
when_to_use_negative:
- Input is short-read bisulfite or EMseq data (WGBS/EMseq) — use conventional short-read
  aligners (e.g. Trim Galore + Bismark) rather than this long-read CT/GA dual-mapping
  pipeline
- You need to distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same
  molecule — nanoEM cannot separate mC from hmC; use direct nanopore WGS signal calling
  instead
- You are running nanopolish methylation calling on R10.4 flowcells — nanopolish does
  not support R10.4 for variant/methylation calling
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: nanoEM pipeline scripts
  role: Core Python3/pysam scripts (convert_ref.py, convert_reads.py, best_align.py,
    call_methylation.py) for reference/read base conversion, dual-alignment selection,
    and CpG methylation frequency calling
  repo: https://github.com/yos-sk/nanoEM
- name: minimap2
  role: Long-read aligner mapping CT- and GA-converted reads to the converted reference
    with -ax map-ont --eqx
- name: samtools
  role: BAM conversion, sorting, indexing, and merging of alignment outputs at each
    pipeline stage
- name: sambamba
  role: mpileup generation restricted to cpg_sites.bed (-t 8 --samtools -f ref) on
    the CT and GA sorted BAMs
- name: nanopolish
  role: Direct signal-level methylation calling from native PromethION WGS data for
    benchmark comparison
  repo: https://github.com/jts/nanopolish
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:39ad21a06bb118e27214d404f93211c5b0ee8812b14f0b289c9d285bc6db3358
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
  - minimap2
  - nanoEM pipeline scripts
  - nanopolish
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nanopore-long-read-methylation-pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A bioinformatics pipeline (nanoEM) that calls per-CpG methylation from enzymatically base-converted (C-to-T) long nanopore reads by dual CT/GA strand-conversion mapping with minimap2, best-alignment selection, and sambamba mpileup-based methylation calling.

## When to use

Use when analyzing EMseq/nanoEM base-converted Oxford Nanopore long reads to quantify per-CpG methylation genome-wide, or when reproducing methylation rates and CpG coverage versus WGBS, EMseq, and nanopolish.

## When NOT to use

- Input is short-read bisulfite or EMseq data (WGBS/EMseq) — use conventional short-read aligners (e.g. Trim Galore + Bismark) rather than this long-read CT/GA dual-mapping pipeline
- You need to distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same molecule — nanoEM cannot separate mC from hmC; use direct nanopore WGS signal calling instead
- You are running nanopolish methylation calling on R10.4 flowcells — nanopolish does not support R10.4 for variant/methylation calling

## Inputs

- nanoEM base-converted nanopore reads (FASTQ) from DDBJ accession DRA011237
- Reference genome FASTA (e.g. GRCh38)
- cpg_sites.bed defining CpG positions for mpileup restriction
- PromethION WGS raw signal data (FAST5/SLOW5) and basecalled reads for nanopolish methylation calling
- Deposited WGBS/EMseq methylation call files from E-GEAD-408 for benchmarking

## Outputs

- output.fa — CT/GA-converted reference genome
- *_CT.fq.gz and *_GA.fq.gz — strand-conversion split reads
- 1.sorted.bam / 2.sorted.bam — dual-conversion alignments
- output_CT.sorted.bam / output_GA.sorted.bam — best-alignment BAMs
- pileup_CT.tsv / pileup_GA.tsv — sambamba mpileup per-CpG pileups
- frequency_methylation.tsv — per-CpG methylation frequencies
- methylation_comparison_table.tsv — per-method methylation rates and covered-CpG counts (>=5 reads)

## How to apply

Convert the reference genome with convert_ref.py (python src/convert_ref.py ref > output.fa) and split nanoEM reads into CT- and GA-converted fastq files with convert_reads.py (python src/convert_reads.py fastq, producing *_CT.fq.gz and *_GA.fq.gz). Align each conversion separately to the converted reference using minimap2 (-t 8 --split-prefix -ax map-ont --eqx), pipe through samtools view/sort/index to obtain 1.sorted.bam and 2.sorted.bam. Select the best alignment across the two strand conversions with best_align.py (--bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq), then re-sort and index output_CT.sam and output_GA.sam. Call per-CpG methylation by running sambamba mpileup on both sorted BAMs restricted to a cpg_sites.bed (-t 8 --samtools -f ref), and combine pileup_CT.tsv and pileup_GA.tsv with call_methylation.py to produce frequency_methylation.tsv; CpG coverage is computed at a >=5 read threshold. Rationale: dual CT/GA mapping resolves the strand ambiguity of C-to-T enzymatic conversion in error-prone ONT reads, and --eqx preserves exact-match CIGAR strings needed for methylation state extraction. For comparison against direct native-DNA calling, process PromethION WGS with nanopolish call-methylation; nanoEM results have been validated against WGBS and EMseq rates (nanoEM 45.7% MB231 / 79.2% BT474).

## Related tools

- **nanoEM pipeline scripts** (Core Python3/pysam scripts (convert_ref.py, convert_reads.py, best_align.py, call_methylation.py) for reference/read base conversion, dual-alignment selection, and CpG methylation frequency calling) — https://github.com/yos-sk/nanoEM
- **minimap2** (Long-read aligner mapping CT- and GA-converted reads to the converted reference with -ax map-ont --eqx)
- **samtools** (BAM conversion, sorting, indexing, and merging of alignment outputs at each pipeline stage)
- **sambamba** (mpileup generation restricted to cpg_sites.bed (-t 8 --samtools -f ref) on the CT and GA sorted BAMs)
- **nanopolish** (Direct signal-level methylation calling from native PromethION WGS data for benchmark comparison) — https://github.com/jts/nanopolish

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam
```

## Evaluation signals

- Reproduced overall CpG methylation rates fall within reasonable tolerance of reported values: nanoEM 45.7% (MB231) / 79.2% (BT474), WGBS 53.3%/55.1% and 82.5%/84.9%, nanopolish 55.5%/79.8%
- frequency_methylation.tsv contains per-CpG methylation frequencies and coverage is computed with a >=5 read threshold
- Both strand-conversion BAMs (output_CT.sorted.bam, output_GA.sorted.bam) are successfully sorted and indexed after best_align.py selection
- minimap2 runs use --eqx CIGAR output; alignments map to the converted reference (output.fa), not the original genome
- Comparison table includes both a methylation-rate column and a covered-CpG-count column for nanoEM, WGBS/EMseq, and nanopolish, reproducing Figures 1A-1C and Supplementary Tables S1, S2, S5

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine (mC) and 5-hydroxymethylcytosine (hmC) on the same DNA molecule, unlike direct nanopore WGS signal calling
- No software version numbers are stated for minimap2, nanopolish, or the pipeline scripts; newer minimap2 (cloned after April 7, 2023) allows dropping --split-prefix
- Direct nanopore WGS requires ~500 ng (150 fmol) of DNA input, whereas nanoEM works from 1 ng but trades off mC/hmC discrimination; nanoEM methylation rates in MB231 were somewhat lower than short-read WGBS/EMseq

## Evidence
