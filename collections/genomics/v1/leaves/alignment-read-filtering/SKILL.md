---
name: alignment-read-filtering
description: Use when a pipeline must pre-partition SV-candidate reads from a BAM/CRAM
  (paired-end bwa-mem or long-read minimap2/ngmlr alignments) before variant calling,
  mirroring dysgu's fetch criteria and depth-skipping rules.
when_to_use_negative:
- Input is already the dysgu temporary reads BAM (temp_dir/input.dysgu_reads.bam)
  — proceed directly to 'dysgu call' instead of re-fetching.
- Reads are unaligned FASTQ or the BAM is not sorted/indexed — fetch requires a sorted,
  indexed alignment file.
- The goal is filtering final SV calls by model probability (>=0.5 PacBio / >=0.35
  ONT) — use 'dysgu filter', not read-level fetch partitioning.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: provides the 'fetch' command that separates SV reads (split/discordant, soft-clipped)
    from the input BAM into a temporary reads file consumed by 'dysgu call'
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:f36ea61bdae2ef36b05c1c14e822e06303a9daca7c5019bac04377d924967767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# alignment-read-filtering

## Summary

Reconstructs dysgu's 'fetch' stage: scan a sorted, indexed BAM/CRAM and partition reads carrying structural-variant signals (MAPQ >=1, soft-clip >=15 bp, discordant insert size/orientation, supplementary alignments, cigar gaps >=30 bp, inter-chromosomal mates) into a temporary SV-reads BAM for downstream calling.

## When to use

Use when a pipeline must pre-partition SV-candidate reads from a BAM/CRAM (paired-end bwa-mem or long-read minimap2/ngmlr alignments) before variant calling, mirroring dysgu's fetch criteria and depth-skipping rules.

## When NOT to use

- Input is already the dysgu temporary reads BAM (temp_dir/input.dysgu_reads.bam) — proceed directly to 'dysgu call' instead of re-fetching.
- Reads are unaligned FASTQ or the BAM is not sorted/indexed — fetch requires a sorted, indexed alignment file.
- The goal is filtering final SV calls by model probability (>=0.5 PacBio / >=0.35 ONT) — use 'dysgu filter', not read-level fetch partitioning.

## Inputs

- sorted, indexed .bam or .cram of paired-end (bwa-mem) or long-read (minimap2/ngmlr) alignments
- working/temp directory for temporary files
- optional indexed reference genome FASTA and insert-size metrics (--ibam all_reads.bam or -I)

## Outputs

- temporary BAM of SV-associated reads (e.g. temp_dir/input.dysgu_reads.bam / candidate_reads.bam, zero-compression level)
- per-read audit table read_criteria_audit.tsv flagging which candidate criteria each read satisfied
- per-region mean-depth statistics marking high-depth (>=200 PE / >=150 LR) skipped regions

## How to apply

Load the input alignment file (e.g. SRA ERR1341796 syndip paired-end reads aligned with bwa-mem) with dysgu's BAM-reading machinery and first compute per-region mean read depth, marking regions with mean depth >=200 for paired-end data (>=150 for long reads) as skipped. Then scan each read and flag it as an SV candidate if it satisfies any dysgu criterion: MAPQ >=1, soft-clip >=15 bp (PE only; the --clip-length default), insert size >= median+5 standard deviations, a supplementary alignment, a cigar gap >=30 bp, or a mate mapped to another chromosome. Write all reads passing any criterion to a temporary BAM (e.g. temp_dir/input.dysgu_reads.bam) written with compression disabled (zero-compression level) to speed I/O, since fetch can generate >5 GB of SV-associated reads for a 30X genome. Simultaneously emit a per-read audit table (read_criteria_audit.tsv) recording which criteria each read satisfied. Validate by confirming candidate_reads.bam is a valid BAM whose read count equals the number of audit-table rows and that recorded criteria match the threshold settings (MAPQ >=1, soft-clip >=15 bp, insert >= median+5 stdev, cigar gap >=30, depth >=200 PE / >=150 LR).

## Related tools

- **dysgu** (provides the 'fetch' command that separates SV reads (split/discordant, soft-clipped) from the input BAM into a temporary reads file consumed by 'dysgu call') — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- candidate_reads.bam exists and is a valid BAM; its read count equals the number of rows in read_criteria_audit.tsv
- recorded criteria in the audit table match dysgu default thresholds: MAPQ >=1, soft-clip >=15 bp, insert size >= median+5 stdev, cigar gap >=30 bp
- high-depth regions (mean depth >=200 PE, >=150 LR) are excluded from the candidate BAM
- BAM is written at zero compression level, so file size is large (>5 GB for a 30X genome) — an unexpectedly tiny output signals a fetch failure
- the temporary BAM is consumable by 'dysgu call' with --ibam insert-size metrics from the full alignment file

## Limitations

- Fetch generates a large temp file of SV-associated reads (>5 GB) for a 30X genome, so disk I/O speed dominates runtime (~1 h, <6 GB memory on one core)
- PE-only criteria (soft-clip >=15 bp, discordant insert size/orientation) do not apply to long-read mode, where depth-skip threshold differs (150 vs 200)
- If insert-size metrics are not supplied via --ibam or -I, dysgu falls back on the input.bam in temp_dir, which may be less accurate
- Over-aggressive 'divergence' settings (default 0.02) can filter too many reads on lower-accuracy data; '--divergence auto' is recommended for unsure read accuracy

## Evidence

- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
- [readme] All SV associated reads will be placed in `samp1_temp/input.dysgu_reads.bam`.: [evidence span withheld — anchor status: external_doc]
- [methods] Compute mean read depth per region and mark regions with mean depth >=200 for paired-end data (>=150 for long reads) as skipped.: [evidence span withheld — anchor status: not_found]
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [readme] Note that when `fetch` is utilized (or using run command), a large temp file is generated consisting of SV-associated reads >5 Gb in size.: [evidence span withheld — anchor status: not_found]
