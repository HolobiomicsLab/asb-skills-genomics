---
name: sv-post-calling-filtering
description: Use when you have a raw NanoSV (or equivalent long-read) SV VCF and need
  to remove likely false positives, especially calls anchored in homopolymer or tandem-repeat
  regions and small indels below the 40 bp SV threshold.
when_to_use_negative:
- Input is raw nanopore FASTQ or an unaligned BAM — NanoSV SV calling (with LAST/BWA-MEM/minimap2/NGMLR
  mapping) must run first to produce the VCF to be filtered
- Goal is detecting small variants or indels <40 bp — this filtering intentionally
  discards sub-40 bp indels because they rarely produce the split alignments NanoSV
  relies on
- Goal is read-based phasing or chromothripsis structure reconstruction — those skills
  operate on phased SNPs and haplotype-separated reads, not on SV call-set curation
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: NanoSV
  role: Produces the initial long-read structural-variant call set (split- and gapped-read
    breakpoint clustering) that this filtering cascade curates
  repo: mroosmalen/nanosv
- name: sambamba
  role: Used to convert, sort, and index the alignment BAM that feeds NanoSV upstream
    of filtering
  repo: biod/sambamba
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:ab90af259a86601bc4d32c87479d819585afc154533b47e24f5706bc84761bd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - sambamba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sv-post-calling-filtering

## Summary

Applies the paper's stringent post-calling filtering cascade to a NanoSV structural-variant call set: first discard SVs whose breakpoints fall in homopolymer/short-tandem-repeat regions, then drop small indels (<40 bp) that typically do not produce split alignments. This conservative filtering trades sensitivity (~75%) for precision (>95%).

## When to use

Use when you have a raw NanoSV (or equivalent long-read) SV VCF and need to remove likely false positives, especially calls anchored in homopolymer or tandem-repeat regions and small indels below the 40 bp SV threshold.

## When NOT to use

- Input is raw nanopore FASTQ or an unaligned BAM — NanoSV SV calling (with LAST/BWA-MEM/minimap2/NGMLR mapping) must run first to produce the VCF to be filtered
- Goal is detecting small variants or indels <40 bp — this filtering intentionally discards sub-40 bp indels because they rarely produce the split alignments NanoSV relies on
- Goal is read-based phasing or chromothripsis structure reconstruction — those skills operate on phased SNPs and haplotype-separated reads, not on SV call-set curation

## Inputs

- NanoSV output VCF of structural-variant calls (e.g. 3957 initial calls for NA12878 chr1 MinION data)
- BED/annotation file of homopolymer and short-tandem-repeat (tandem-repeat) regions for the reference genome
- Coordinate-sorted, indexed BAM of long-read alignments (the upstream NanoSV input, needed if re-calling is required)

## Outputs

- Intermediate repeat-filtered VCF (expected 657 calls for the NA12878 chr1 benchmark)
- Final filtered VCF, nanosv_na12878_filtered.vcf, with homopolymer/STR-overlapping calls and <40 bp indels removed (expected 654 calls)

## How to apply

Parse the initial NanoSV VCF (for NA12878 chromosome 1 nanopore data this contains 3957 structural-variant calls with per-variant coordinates, type, and size). Intersect each SV's breakpoint coordinates with homopolymer and tandem-repeat region annotations and discard every call whose breakpoint falls inside such a repeat region, keeping the intermediate call set of 657 SVs as an intermediate VCF. From the repeat-filtered set, exclude indel calls smaller than the 40 bp threshold — below this size variants do not typically result in a split alignment, which is what NanoSV detects via its split-read/gapped-read clustering. Write the final 654-call set to nanosv_na12878_filtered.vcf and verify row counts at each stage. Rationale: the paper deliberately applies 'extremely stringent filtering' to match precision benchmarks (>95%) while accepting an estimated sensitivity of ~75%.

## Related tools

- **NanoSV** (Produces the initial long-read structural-variant call set (split- and gapped-read breakpoint clustering) that this filtering cascade curates) — mroosmalen/nanosv
- **sambamba** (Used to convert, sort, and index the alignment BAM that feeds NanoSV upstream of filtering) — biod/sambamba

## Evaluation signals

- Repeat-filter stage must retain exactly 657 calls from the initial 3957 NA12878 chr1 calls
- Indel-size filter must produce a final call set of exactly 654 calls, matching the paper's reported cascade
- Verify VCF row counts after each filter stage against the reported 657 and 654 values
- Assert no retained SV breakpoint lies within a homopolymer or tandem-repeat region annotation
- Assert every retained indel is ≥40 bp (smaller indels excluded as they do not typically yield split alignments)

## Limitations

- Extremely stringent post-calling filtering reduces estimated sensitivity to ~75%, so true SVs will be discarded
- The conservative SV calling and post-calling filtering steps lower the fraction of novel variants relative to other long-read studies
- SV detection in complex repeat regions and segmental duplications remains an unresolved challenge, so breakpoint-anchored repeat filtering may not capture all error modes

## Evidence

- [discussion] Stringent filtering trades sensitivity for precision: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] Conservative calling/filtering lower novel-variant fraction: "this is partly due to our conservative SV calling and post-calling filtering steps"
- [readme] NanoSV detects SVs from split/gapped alignments, explaining the 40 bp indel cutoff: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
- [discussion] Repeat-region SV catalogs are an open challenge: "Creating larger catalogs of SVs, in complex repeat regions and segmental duplications, is a particular challenge in the coming years"
