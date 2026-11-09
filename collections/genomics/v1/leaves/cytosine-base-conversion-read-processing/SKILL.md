---
name: cytosine-base-conversion-read-processing
description: Use when analyzing nanoEM/EMseq-converted nanopore reads (DRA011237-style
  fastq from PromethION) to call per-CpG methylation and compute overall CpG methylation
  rates and CpG coverage for comparison with WGBS, EMseq, or nanopolish calls.
when_to_use_negative:
- Input is native (unconverted) nanopore WGS signal data intended for direct methylation
  calling with nanopolish rather than enzymatically converted reads
- Input reads are short-read bisulfite (WGBS/EMseq) data that require conventional
  short-read bisulfite aligners, not ONT-oriented minimap2 map-ont alignment
- Nanopore library uses R10.4 flowcells where nanopolish documentation notes variant/methylation
  calling support is absent
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns CT- and GA-converted long reads to the converted reference using the
    map-ont preset with --eqx CIGAR
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Sorts, indexes, converts, and merges BAM/SAM alignment outputs throughout
    the pipeline
- name: sambamba
  role: Runs mpileup restricted to cpg_sites.bed on both CT and GA sorted BAMs for
    methylation calling
- name: pysam
  role: Python library underlying the pipeline scripts convert_ref.py, convert_reads.py,
    best_align.py, and call_methylation.py
  repo: https://github.com/yos-sk/nanoEM
- name: nanopolish
  role: Provides native-DNA methylation calls (call-methylation) used as the direct-calling
    benchmark for comparison
  repo: https://github.com/jts/nanopolish
- name: TrimGalore
  role: Adapter/quality trimming for associated short-read bisulfite comparisons
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:1c4d26451f656edbce32008802ce97b64730e7bb641f3ef4dd728e4a7d6e02d9
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
  - TrimGalore
  - minimap2
  - nanopolish
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cytosine-base-conversion-read-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process methylation-sensitive enzymatically base-converted (C-to-T) nanopore long reads for whole-genome CpG methylation calling using the nanoEM pipeline, which dual-converts reads into CT and GA fastq representations and resolves best alignments against a converted reference.

## When to use

Use when analyzing nanoEM/EMseq-converted nanopore reads (DRA011237-style fastq from PromethION) to call per-CpG methylation and compute overall CpG methylation rates and CpG coverage for comparison with WGBS, EMseq, or nanopolish calls.

## When NOT to use

- Input is native (unconverted) nanopore WGS signal data intended for direct methylation calling with nanopolish rather than enzymatically converted reads
- Input reads are short-read bisulfite (WGBS/EMseq) data that require conventional short-read bisulfite aligners, not ONT-oriented minimap2 map-ont alignment
- Nanopore library uses R10.4 flowcells where nanopolish documentation notes variant/methylation calling support is absent

## Inputs

- nanoEM nanopore fastq reads (e.g. from DDBJ DRA011237)
- Reference genome FASTA (unconverted)
- cpg_sites.bed restricting pileup to CpG positions
- nanopolish methylation call files (from E-GEAD-408) for cross-method comparison
- Deposited WGBS/EMseq methylation calls (optional benchmark)

## Outputs

- output.fa (converted reference genome FASTA)
- *_CT.fq.gz and *_GA.fq.gz (strand-converted read fastqs)
- output_CT.sorted.bam and output_GA.sorted.bam (best-alignment BAMs)
- pileup_CT.tsv and pileup_GA.tsv (sambamba mpileup per-CpG pileups)
- frequency_methylation.tsv (per-CpG methylation frequencies)
- methylation_comparison_table.tsv (per-method methylation rates and covered-CpG counts)

## How to apply

First convert the reference genome with convert_ref.py (python src/convert_ref.py ref > output.fa) and split each nanoEM read set into C-to-T and G-to-A converted fastq files with convert_reads.py, producing *_CT.fq.gz and *_GA.fq.gz. Align both strand-conversions to the converted reference with minimap2 using '-t 8 --split-prefix -ax map-ont --eqx' (map-ont preset preserves sensitivity on error-prone ONT data; --eqx retains exact-match CIGAR needed for methylation-aware pileup), then sort/index the BAMs with samtools. Select best alignments across the CT and GA BAMs with best_align.py (--bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq), which resolves the original strand, then re-sort output_CT.sam and output_GA.sam into indexed BAMs. Call per-CpG methylation by running sambamba mpileup on both sorted BAMs restricted to a cpg_sites.bed BED file (-t 8 --samtools -f ref), producing pileup_CT.tsv and pileup_GA.tsv, then combine them with call_methylation.py > frequency_methylation.tsv. Compute CpG coverage at a threshold of >=5 reads per CpG, and optionally produce IGV bisulfite-mode visualization BAMs with vis_GA_utilities.py and samtools merge.

## Related tools

- **minimap2** (Aligns CT- and GA-converted long reads to the converted reference using the map-ont preset with --eqx CIGAR) — https://github.com/yos-sk/nanoEM
- **samtools** (Sorts, indexes, converts, and merges BAM/SAM alignment outputs throughout the pipeline)
- **sambamba** (Runs mpileup restricted to cpg_sites.bed on both CT and GA sorted BAMs for methylation calling)
- **pysam** (Python library underlying the pipeline scripts convert_ref.py, convert_reads.py, best_align.py, and call_methylation.py) — https://github.com/yos-sk/nanoEM
- **nanopolish** (Provides native-DNA methylation calls (call-methylation) used as the direct-calling benchmark for comparison) — https://github.com/jts/nanopolish
- **TrimGalore** (Adapter/quality trimming for associated short-read bisulfite comparisons) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
python src/convert_reads.py fastq && minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam
```

## Evaluation signals

- Reproduced overall CpG methylation rates match reported values within tolerance: nanoEM 45.7% (MB231) / 79.2% (BT474), WGBS 53.3%/55.1% and 82.5%/84.9%, nanopolish 55.5%/79.8%
- methylation_comparison_table.tsv contains a methylation-rate column and a covered-CpG-count column for each of nanoEM, WGBS/EMseq, and nanopolish
- CpG coverage is computed at the >=5 reads threshold and covered-CpG counts are consistent with Supplementary Tables S1, S2, S5 values
- Both *_CT.fq.gz and *_GA.fq.gz fastq sets exist and each produces a non-empty indexed sorted BAM after best_align.py selection
- frequency_methylation.tsv is generated only after both pileup_CT.tsv and pileup_GA.tsv are combined by call_methylation.py

## Limitations

- nanoEM cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling
- No version numbers are specified for minimap2, nanopolish, or the pipeline software; --split-prefix behavior depends on the minimap2 build date (post April 7, 2023)
- nanoEM's overall methylation rate in MB231 (45.7%) was somewhat lower than short-read WGBS (53.3%), so cross-method agreement is approximate, not exact

## Evidence

- [results] nanoEM's overall CpG methylation rates (45.7% MB231, 79.2% BT474) were almost consistent with short-read WGBS and EMseq, though nanoEM in MB231 was somewhat lower.: [evidence span withheld — anchor status: not_found]
- [readme] OUTPUT:
- *_CT.fq.gz: C-to-T converted fastq
- *_GA.fq.gz: G-to-A converted fastq: "*_CT.fq.gz: C-to-T converted fastq"
