---
name: insert-size-outlier-detection
description: Use when reconstructing or reimplementing the read-selection stage of
  a paired-end SV caller (e.g. dysgu fetch) and you need to classify which reads are
  SV candidates based on insert-size outliers, soft-clips (≥15 bp), MAPQ (≥1), cigar
  gaps (≥30 bp), or inter-chromosomal mates.
when_to_use_negative:
- Input is long-read data — the insert-size/soft-clip paired-end criteria and the
  ≥200 depth cutoff do not apply (long reads use a ≥150 depth cutoff and discordant-pair
  signals are not relevant)
- Reads are unsorted or the BAM lacks an index — dysgu's fetch expects a sorted and
  indexed .bam/.cram
- The goal is variant-level (not read-level) filtering — downstream call filtering
  uses model probability thresholds (≥0.5 PacBio, ≥0.35 ONT), not insert-size outlier
  read flags
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the 'fetch' command that separates SV reads (split/discordant, soft-clipped
    ≥15 bp, insert-size outliers) from the input BAM into a temporary BAM for the
    call stage
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:2f449b098e3e8f685d978bc8afe574d6c657517250305cb47d8ffc45f721f737
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# insert-size-outlier-detection

## Summary

Flags paired-end reads as SV candidates when their inferred insert size is discordant — an extreme outlier relative to the sample's insert-size distribution (≥ median + 5 standard deviations) — as part of dysgu's 'fetch' stage that partitions SV-associated reads into a temporary BAM.

## When to use

Use when reconstructing or reimplementing the read-selection stage of a paired-end SV caller (e.g. dysgu fetch) and you need to classify which reads are SV candidates based on insert-size outliers, soft-clips (≥15 bp), MAPQ (≥1), cigar gaps (≥30 bp), or inter-chromosomal mates.

## When NOT to use

- Input is long-read data — the insert-size/soft-clip paired-end criteria and the ≥200 depth cutoff do not apply (long reads use a ≥150 depth cutoff and discordant-pair signals are not relevant)
- Reads are unsorted or the BAM lacks an index — dysgu's fetch expects a sorted and indexed .bam/.cram
- The goal is variant-level (not read-level) filtering — downstream call filtering uses model probability thresholds (≥0.5 PacBio, ≥0.35 ONT), not insert-size outlier read flags

## Inputs

- Sorted, indexed BAM/CRAM of paired-end reads aligned with bwa-mem (e.g. SRA ERR1341796 syndip reads)
- Insert-size metrics inferred from the main alignment file (via --ibam) or specified manually with -I
- dysgu default argument threshold settings (MAPQ, clip_length, insert-size stdev multiplier, cigar gap, depth cutoffs)

## Outputs

- candidate_reads.bam — temporary BAM of all SV-candidate reads (zero-compression level)
- read_criteria_audit.tsv — per-read table recording which candidate criteria (insert size, soft-clip, MAPQ, cigar gap, mate-chromosome) were satisfied

## How to apply

First infer insert-size metrics from the main paired-end alignment file (aligned e.g. with bwa-mem), since dysgu recommends passing --ibam to the call stage for accurate metrics; if unavailable it falls back to the input.bam in the temp folder, which may be less accurate. Scan each read and flag it as an SV candidate if its insert size is ≥ median + 5 standard deviations of the sample distribution, or if it meets any parallel dysgu criterion: MAPQ ≥1, soft-clip ≥15 bp (paired-end only), a supplementary alignment, a cigar gap ≥30 bp, or a mate mapped to another chromosome. Before scanning, compute mean read depth per region and skip regions with depth ≥200 for paired-end data (≥150 for long reads). Write all reads passing any criterion to a temporary BAM (input.dysgu_reads.bam) written with compression disabled (zero-compression level), and emit a per-read audit table (read_criteria_audit.tsv) recording which criteria each read satisfied.

## Related tools

- **dysgu** (Provides the 'fetch' command that separates SV reads (split/discordant, soft-clipped ≥15 bp, insert-size outliers) from the input BAM into a temporary BAM for the call stage) — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- candidate_reads.bam exists and is a valid BAM whose read count equals the number of rows in read_criteria_audit.tsv
- Criteria recorded per read in the audit table match the dysgu_default_args thresholds (MAPQ ≥1, soft-clip ≥15 bp, insert ≥ median+5 stdev, cigar gap ≥30, depth ≥200 PE / ≥150 LR)
- Skipped regions have mean depth ≥200 (paired-end) or ≥150 (long reads) as computed per region
- The temporary BAM was written with compression disabled (zero-compression level)
- Running 'dysgu fetch samp1_temp input.bam' independently reproduces samp1_temp/input.dysgu_reads.bam consistent with the run pipeline's fetch stage

## Limitations

- A large temporary file of SV-associated reads (>5 GB) is generated when fetch is used, requiring disk space
- Insert-size metrics from the input.bam in the temp folder (when --ibam is not provided) may be less accurate than metrics inferred from the full alignment file
- The depth cutoff for skipping regions (mean depth ≥200 PE / ≥150 LR) is fixed by default arguments and may not suit extreme-coverage samples

## Evidence

- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
- [readme] All SV associated reads will be placed in `samp1_temp/input.dysgu_reads.bam`.: [evidence span withheld — anchor status: external_doc]
- [readme] Additionally, the `--ibam` option is recommended for paired-end data so dysgu can infer insert size metrics from the main alignment file.: "Additionally, the `--ibam` option is recommended for paired-end data so dysgu can infer insert
size metrics from the main alignment file."
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [readme] Also note that when `fetch` is utilized (or using run command), a large temp file is generated consisting of SV-associated reads >5 Gb in size.: [evidence span withheld — anchor status: external_doc]
