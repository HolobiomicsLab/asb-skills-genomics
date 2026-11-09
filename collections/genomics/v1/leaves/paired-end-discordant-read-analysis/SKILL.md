---
name: paired-end-discordant-read-analysis
description: Use when a paired-end (e.g. bwa-mem aligned syndip/GIAB) or long-read
  alignment file must be pre-filtered into SV-associated reads, or when auditing which
  read-level criteria (MAPQ, soft-clip, insert size) selected each read.
when_to_use_negative:
- Input is raw unaligned reads (FASTQ/BAM lacking alignment, MAPQ, CIGAR, or mate
  information) — the criteria require aligned reads
- Input is already a filtered SV-read BAM or a dysgu call-set VCF — the fetch stage
  would re-partition already-selected reads
- Goal is post-calling variant filtering (e.g. model probability >=0.5 PacBio / >=0.35
  ONT) — that is dysgu's separate 'filter' command operating on calls, not reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Provides the 'fetch' command that separates SV-associated reads from the input
    BAM into a temporary BAM using MAPQ, soft-clip, insert-size, cigar-gap, and mate-chromosome
    criteria
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:e96ae5e6127f0a809c44f0cecd36c62481f221eddb8b519e0ec3d685edfb986b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# paired-end-discordant-read-analysis

## Summary

Reconstructs dysgu's 'fetch' stage, which scans a sorted/indexed BAM/CRAM and partitions SV-candidate reads (split, discordant, soft-clipped, supplementary-aligned) into a temporary BAM that feeds the downstream 'call' stage.

## When to use

Use when a paired-end (e.g. bwa-mem aligned syndip/GIAB) or long-read alignment file must be pre-filtered into SV-associated reads, or when auditing which read-level criteria (MAPQ, soft-clip, insert size) selected each read.

## When NOT to use

- Input is raw unaligned reads (FASTQ/BAM lacking alignment, MAPQ, CIGAR, or mate information) — the criteria require aligned reads
- Input is already a filtered SV-read BAM or a dysgu call-set VCF — the fetch stage would re-partition already-selected reads
- Goal is post-calling variant filtering (e.g. model probability >=0.5 PacBio / >=0.35 ONT) — that is dysgu's separate 'filter' command operating on calls, not reads

## Inputs

- Sorted and indexed BAM/CRAM of paired-end reads (e.g. SRA ERR1341796 syndip reads aligned with bwa-mem) or long reads aligned with minimap2/ngmlr
- Indexed reference genome FASTA
- Working/temp directory (temp_dir) for intermediate files

## Outputs

- Temporary BAM of SV-associated reads (e.g. temp_dir/input.dysgu_reads.bam, written at zero compression level, typically >5 Gb for a 30X genome)
- Per-read audit table read_criteria_audit.tsv (one row per candidate read, with satisfied criteria flags)
- Per-region mean-depth annotations marking regions skipped at mean depth >=200 (PE) / >=150 (LR)

## How to apply

Load a sorted, indexed input BAM/CRAM with dysgu's BAM-reading machinery, compute mean read depth per region, and mark regions with mean depth >=200 (paired-end) or >=150 (long reads) as skipped. Scan every read and flag it as an SV candidate if it satisfies any dysgu criterion: MAPQ >=1, soft-clip >=15 bp (paired-end only), insert size >= median + 5 standard deviations (paired-end only), a supplementary alignment, a cigar gap >=30 bp, or a mate mapped to another chromosome. Write all reads passing any criterion to a temporary BAM (e.g. temp_dir/input.dysgu_reads.bam) written with compression disabled (zero compression level), and emit a per-read audit table (read_criteria_audit.tsv) recording which criteria each read satisfied. Rationale: 'fetch' is the first stage of 'dysgu run' and its thresholds — clip_length 15 bp by default for paired-end reads — control the recall ceiling of the whole SV caller, since 'call' only analyses reads in the temporary BAM.

## Related tools

- **dysgu** (Provides the 'fetch' command that separates SV-associated reads from the input BAM into a temporary BAM using MAPQ, soft-clip, insert-size, cigar-gap, and mate-chromosome criteria) — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- candidate_reads.bam exists and is a valid BAM whose read count equals the number of rows in read_criteria_audit.tsv
- Recorded criteria in the audit table match the dysgu_default_args thresholds: MAPQ >=1, soft-clip >=15 bp, insert >= median+5 stdev, cigar gap >=30 bp
- Skipped regions correspond to mean depth >=200 for paired-end data (>=150 for long reads)
- Output temporary BAM is written with compression disabled (zero compression level)
- Downstream 'dysgu call' can consume the temporary BAM via --ibam to infer insert-size metrics from the main alignment file

## Limitations

- For a 30X coverage genome, fetch generates a large temporary file of SV-associated reads >5 Gb, so disk speed and space matter
- Soft-clip and discordant-pair criteria are paired-end specific; long-read candidates rely on supplementary alignments, cigar gaps, and depth limits
- Dysgu's heuristics (e.g. span position distance, minimizer support) are unoptimized and could be tuned with a larger test set

## Evidence

- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
- [readme] All SV associated reads will be placed in `samp1_temp/input.dysgu_reads.bam`.: [evidence span withheld — anchor status: external_doc]
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [intro] Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings.: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
