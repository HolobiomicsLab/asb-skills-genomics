---
name: structural-variant-candidate-detection
description: Use when you have a sorted, indexed .bam/.cram (bwa-mem paired-end or
  minimap2/ngmlr long reads) and need to partition reads carrying SV signals (soft-clips
  >=15 bp, discordant pairs, supplementary mappings, cigar gaps) before SV calling.
when_to_use_negative:
- You already have a filtered SV VCF and want post-calling filtering — use 'dysgu
  filter' instead of re-running fetch.
- Input reads are unaligned FASTQ with no BAM/CRAM — fetch operates on sorted, indexed
  alignments, so an aligner (bwa-mem, minimap2, ngmlr) must be run first.
- You need full-length insertion resolution or multi-allelic site resolution in PE
  data — the README notes dysgu lacks a de novo assembly stage for this.
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dysgu
  role: Provides the 'fetch' command (also wrapped by 'run') that scans the alignment
    file and writes candidate SV reads to input.dysgu_reads.bam; presets --mode pacbio-*
    / nanopore-* apply for long-read BAMs.
  repo: https://github.com/kcleal/dysgu
- name: samtools
  role: Streams selected BAM regions (e.g. samtools view -bh samp.bam chr1:0-1000000)
    to dysgu run/fetch via stdin.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:6d3a8e527cfc2fc23592824850e65bb8c8117cca2475c0bfffb1e059d9b3d3b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - samtools
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# structural-variant-candidate-detection

## Summary

Isolate SV-associated reads (split, discordant, soft-clipped, cigar-gap reads) from a sorted/indexed BAM or CRAM using dysgu's 'fetch' stage, producing a temporary dysgu_reads BAM that feeds the 'call' stage of dysgu's PE/LR structural variant calling pipeline.

## When to use

Use when you have a sorted, indexed .bam/.cram (bwa-mem paired-end or minimap2/ngmlr long reads) and need to partition reads carrying SV signals (soft-clips >=15 bp, discordant pairs, supplementary mappings, cigar gaps) before SV calling.

## When NOT to use

- You already have a filtered SV VCF and want post-calling filtering — use 'dysgu filter' instead of re-running fetch.
- Input reads are unaligned FASTQ with no BAM/CRAM — fetch operates on sorted, indexed alignments, so an aligner (bwa-mem, minimap2, ngmlr) must be run first.
- You need full-length insertion resolution or multi-allelic site resolution in PE data — the README notes dysgu lacks a de novo assembly stage for this.

## Inputs

- Sorted and indexed .bam or .cram of reads aligned with bwa-mem (paired-end) or minimap2/ngmlr (long reads)
- Indexed reference genome in FASTA format
- Working/temporary directory (e.g. temp_dir) for fetch outputs
- Optional full read alignment BAM via --ibam for insert-size inference (PE data)

## Outputs

- temp_dir/input.dysgu_reads.bam — temporary BAM of SV-associated (candidate) reads
- Additional analysis files in temp_dir consumed by dysgu call
- Downstream svs.vcf after the call stage (dysgu run > svs.vcf)

## How to apply

Run dysgu fetch (or dysgu run, which wraps fetch + call) on a sorted/indexed BAM plus indexed reference FASTA, supplying a working directory for temporary files. Fetch scans each read and retains it as an SV candidate if it shows a soft-clip >= clip_length (15 bp by default for paired-end reads), a discordant insert size/orientation, a supplementary alignment, a cigar gap, or a mate mapped to another chromosome; high-depth regions (mean depth >=200 for PE, >=150 for long reads) are skipped. Candidate reads are written to temp_dir/input.dysgu_reads.bam, with a large temp file (>5 Gb) expected for a 30X WGA PE genome. For paired-end data, later pass --ibam to the call stage so insert-size metrics are inferred from the full alignment file rather than the smaller fetched BAM, or specify insert size manually with -I.

## Related tools

- **dysgu** (Provides the 'fetch' command (also wrapped by 'run') that scans the alignment file and writes candidate SV reads to input.dysgu_reads.bam; presets --mode pacbio-* / nanopore-* apply for long-read BAMs.) — https://github.com/kcleal/dysgu
- **samtools** (Streams selected BAM regions (e.g. samtools view -bh samp.bam chr1:0-1000000) to dysgu run/fetch via stdin.)

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- temp_dir/input.dysgu_reads.bam exists and is a valid BAM containing only SV-signal reads (soft-clips >=15 bp PE, discordant pairs, supplementary alignments, cigar gaps)
- Candidate read set is consistent with the default thresholds: clip_length 15 bp for PE, insert size >= median+5 stdev, cigar gap >=30 bp, MAPQ >=1
- For a 30X 150 bp PE genome on one core, the temp dysgu_reads file exceeds ~5 Gb and total memory stays <6 GB
- The subsequent 'call' stage accepts the temp BAM (with --ibam all_reads.bam for accurate insert-size metrics) and emits a well-formed svs.vcf
- Running 'dysgu test' passes, confirming a functional install

## Limitations

- Fetch generates a large temporary file of SV-associated reads (>5 Gb for a 30X genome), so disk throughput and space matter.
- Without --ibam, insert-size metrics are inferred from the smaller dysgu_reads BAM, which 'may be less accurate'; manual -I is the alternative.
- Soft-clip candidate detection uses clip_length 15 bp by default for paired-end reads; divergence default 0.02 can be too stringent for lower-accuracy reads (use --divergence auto) and will result in more reads being filtered and lower sensitivity.

## Evidence

- [readme] fetch scans alignment for candidate reads: [evidence span withheld — anchor status: external_doc]
- [readme] candidate reads output location: [evidence span withheld — anchor status: external_doc]
- [intro] conceptual basis for candidate selection: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
