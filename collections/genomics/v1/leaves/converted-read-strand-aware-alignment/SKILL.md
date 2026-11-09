---
name: converted-read-strand-aware-alignment
description: Use when input is base-converted (EMseq/nanoEM) nanopore FASTQ split
  into *_CT.fq.gz and *_GA.fq.gz and you need strand-aware mapping to a converted
  reference before methylation calling.
when_to_use_negative:
- Input reads are native (unconverted) nanopore WGS to be methylation-called directly
  with nanopolish — no strand conversion or converted reference is needed
- Input is short-read WGBS/EMseq data aligned with bisulfite-aware short-read aligners
  rather than long-read nanopore data
- The reference genome has not been base-converted with convert_ref.py; aligning converted
  reads to an unconverted reference will fail to map correctly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns CT- and GA-converted nanopore reads to the converted reference using
    '-ax map-ont --eqx' with -t 8 and --split-prefix
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Views, sorts and indexes the minimap2 output BAMs and the best-alignment SAMs
  repo: https://github.com/yos-sk/nanoEM
- name: pysam
  role: Python library used by best_align.py to select the best alignment across the
    two strand conversions
  repo: https://github.com/yos-sk/nanoEM
- name: sambamba
  role: Runs mpileup restricted to cpg_sites.bed on the sorted best-alignment BAMs
    for downstream methylation calling
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:17274b674ec43ac2334336ea3fba3b2b5ed2730c5b730f92b4a455e5f6f7ddd6
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
  - nanopolish
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# converted-read-strand-aware-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns C-to-T and G-to-A converted nanoEM nanopore reads to a correspondingly converted reference genome with minimap2 (--eqx, map-ont preset) and selects the best alignment across the two strand conversions, enabling per-CpG methylation calling from enzymatically base-converted long reads.

## When to use

Use when input is base-converted (EMseq/nanoEM) nanopore FASTQ split into *_CT.fq.gz and *_GA.fq.gz and you need strand-aware mapping to a converted reference before methylation calling.

## When NOT to use

- Input reads are native (unconverted) nanopore WGS to be methylation-called directly with nanopolish — no strand conversion or converted reference is needed
- Input is short-read WGBS/EMseq data aligned with bisulfite-aware short-read aligners rather than long-read nanopore data
- The reference genome has not been base-converted with convert_ref.py; aligning converted reads to an unconverted reference will fail to map correctly

## Inputs

- C-to-T converted nanopore FASTQ (*_CT.fq.gz)
- G-to-A converted nanopore FASTQ (*_GA.fq.gz)
- Base-converted reference genome FASTA (output.fa from convert_ref.py)
- Sorted BAMs from both strand-conversion alignments (1.sorted.bam, 2.sorted.bam)
- Original (unconverted) FASTQ passed to best_align.py

## Outputs

- Best-alignment SAM files per strand conversion (output_CT.sam, output_GA.sam)
- Sorted, indexed best-alignment BAMs (output_CT.sorted.bam, output_GA.sorted.bam)
- Per-CpG pileup files (pileup_CT.tsv, pileup_GA.tsv) feeding call_methylation.py

## How to apply

First convert the reference genome (convert_ref.py) and the reads (convert_reads.py), producing *_CT.fq.gz (C-to-T) and *_GA.fq.gz (G-to-A) fastq files. Align each converted fastq separately to the converted reference using minimap2 with the ONT preset '-ax map-ont --eqx' and -t 8, then view/sort/index with samtools into 1.sorted.bam and 2.sorted.bam. Run best_align.py with --bam1/--bam2 pointing at the two sorted BAMs plus the original fastq to pick the better alignment per read across the two strand conversions, then sort and index output_CT.sam and output_GA.sam into output_CT.sorted.bam and output_GA.sorted.bam. The rationale is that base conversion creates strand-specific reference mismatches that ordinary alignment would score poorly, so both conversion directions must be mapped and compared per read; --eqx preserves exact-match/ mismatch tags needed for methylation calls downstream via sambamba mpileup and call_methylation.py.

## Related tools

- **minimap2** (Aligns CT- and GA-converted nanopore reads to the converted reference using '-ax map-ont --eqx' with -t 8 and --split-prefix) — https://github.com/yos-sk/nanoEM
- **samtools** (Views, sorts and indexes the minimap2 output BAMs and the best-alignment SAMs) — https://github.com/yos-sk/nanoEM
- **pysam** (Python library used by best_align.py to select the best alignment across the two strand conversions) — https://github.com/yos-sk/nanoEM
- **sambamba** (Runs mpileup restricted to cpg_sites.bed on the sorted best-alignment BAMs for downstream methylation calling) — https://github.com/yos-sk/nanoEM

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa sample_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam && python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq sample.fq.gz
```

## Evaluation signals

- Both strand conversions produce a sorted, indexed BAM (1.sorted.bam, 2.sorted.bam) and best_align.py emits output_CT.sam and output_GA.sam that each sort and index without error
- minimap2 runs use the --eqx flag so alignment records contain exact-match (=) and mismatch (X) CIGAR operators required for methylation extraction
- Downstream call_methylation.py accepts both pileup_CT.tsv and pileup_GA.tsv and produces frequency_methylation.tsv with per-CpG methylation frequencies
- Reproduced overall CpG methylation rates approximate the reported values (nanoEM 45.7% MB231 / 79.2% BT474) within reasonable tolerance
- Using the --split-prefix option is unnecessary (and faster) when using a minimap2 version obtained via git clone after April 7, 2023

## Limitations

- No version numbers are given for minimap2, samtools, sambamba or the pipeline scripts, so exact reproducibility of alignment behavior is not documented
- nanoEM cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling
- The pipeline analyzes reads less efficiently at CGIs than a purely minimap2-based prior approach implies overhead from the dual strand-conversion mapping

## Evidence

- [readme] minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx: "minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx"
- [readme] best_align.py selects best alignments across the two strand conversions: "python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam  --fastq fastq"
