---
name: cigar-alignment-gap-parsing
description: Use when starting from a sorted, indexed BAM/CRAM of paired-end (bwa-mem)
  or long reads (minimap2/ngmlr) and you need to isolate SV-supporting reads via CIGAR
  gaps, soft-clips, or discordant mappings before SV calling.
when_to_use_negative:
- Input is already a VCF of called SVs — use dysgu 'filter' or 'merge' instead of
  re-fetching reads
- Long-read data with high error rates where default thresholds (soft-clip 15 bp,
  divergence 0.02) would over-filter — set '--divergence auto' and appropriate --mode
  first
- Reads lacking an index or unsorted — fetch requires a sorted, indexed .bam/cram
  plus indexed reference
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: dysgu
  role: Provides the 'fetch' command that parses CIGAR gaps, soft-clips, and discordant/supplementary
    mappings to partition SV-candidate reads into a temporary BAM
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:7f3032e844665340022f85e7c481deefdcac043a46e4ffd8723582eef19ca312
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cigar-alignment-gap-parsing

## Summary

Scans alignment CIGAR strings and mapping metadata in a BAM/CRAM to flag individual reads as structural-variant candidates — the read-partitioning logic of dysgu's 'fetch' stage — producing a temporary BAM of SV-associated reads for downstream calling.

## When to use

Use when starting from a sorted, indexed BAM/CRAM of paired-end (bwa-mem) or long reads (minimap2/ngmlr) and you need to isolate SV-supporting reads via CIGAR gaps, soft-clips, or discordant mappings before SV calling.

## When NOT to use

- Input is already a VCF of called SVs — use dysgu 'filter' or 'merge' instead of re-fetching reads
- Long-read data with high error rates where default thresholds (soft-clip 15 bp, divergence 0.02) would over-filter — set '--divergence auto' and appropriate --mode first
- Reads lacking an index or unsorted — fetch requires a sorted, indexed .bam/cram plus indexed reference

## Inputs

- Sorted and indexed BAM/CRAM of aligned paired-end reads (e.g. bwa-mem) or long reads (minimap2/ngmlr), optionally streamed via stdin
- Indexed reference genome FASTA
- Working/temporary directory for dysgu temp files

## Outputs

- Temporary BAM of SV-associated reads (e.g. samp1_temp/input.dysgu_reads.bam, written with zero compression)
- Insert-size metrics / candidates passed to the 'call' stage (consumed via dysgu call --ibam)
- Per-read audit of which candidate criteria each read satisfied (e.g. read_criteria_audit.tsv in reconstructed workflows)

## How to apply

Run dysgu's fetch stage over the input alignment: compute mean read depth per region (regions with mean depth >=200 for paired-end data, >=150 for long reads, are skipped), then scan each read and flag it as an SV candidate if it satisfies any criterion — MAPQ >=1, soft-clip >=15 bp (paired-end default via --clip_length), insert size >= median+5 standard deviations (inferred from the main BAM with --ibam), a supplementary alignment, a CIGAR alignment gap >=30 bp, or a mate mapped to another chromosome. Write all reads passing any criterion to a temporary BAM (dysgu_reads.bam, written with compression disabled) in the provided working directory. The rationale, per the article, is that dysgu 'identif[ies] SVs from alignment cigar information as well as discordant and split-read mappings', so capturing gap- and clip-bearing reads at this stage is what feeds the consensus-contig and machine-learning classification stages. For long reads, choose the --mode preset (e.g. pacbio-revio, nanopore-r10) and consider '--divergence auto' since the default 0.02 divergence can over-filter lower-accuracy reads.

## Related tools

- **dysgu** (Provides the 'fetch' command that parses CIGAR gaps, soft-clips, and discordant/supplementary mappings to partition SV-candidate reads into a temporary BAM) — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- candidate_reads.bam exists and is a valid BAM whose read count equals the number of rows in the per-read audit table (read_criteria_audit.tsv)
- Every read in the output BAM satisfies at least one criterion: MAPQ >=1, soft-clip >=15 bp, insert size >= median+5 SD, supplementary alignment, CIGAR gap >=30 bp, or inter-chromosomal mate
- Skipped regions match the depth caps: mean depth >=200 (paired-end) or >=150 (long reads) excluded
- Recorded criteria are consistent with the dysgu default threshold settings used for the run
- Output temp BAM is large (fetch generates an SV-associated read temp file >5 Gb for a 30X genome), confirming reads were actually captured

## Limitations

- Soft-clip and discordant-pair criteria apply to paired-end data only (15 bp clip default for PE reads)
- High-depth regions (mean depth >=200 PE / >=150 LR) are skipped entirely and yield no candidates
- For long reads with higher error rates, the default 0.02 divergence can be too stringent, filtering more reads and lowering sensitivity
- CIGAR/clip heuristics such as span position distance and minimizer support are acknowledged as unoptimized and potentially tunable

## Evidence

- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [intro] Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings.: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
- [readme] All SV associated reads will be placed in `samp1_temp/input.dysgu_reads.bam`.: [evidence span withheld — anchor status: external_doc]
- [readme] when `fetch` is utilized (or using run command), a large temp file is generated consisting of SV-associated reads >5 Gb in size.: [evidence span withheld — anchor status: external_doc]
