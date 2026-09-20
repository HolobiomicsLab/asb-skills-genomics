---
name: minimap2-overlap-detection
description: Use when long noisy reads (PacBio CLR or Nanopore R9/R10) need candidate
  overlaps for string-graph construction, or when corrected reads must be mapped to
  contigs with base-level alignment for SNP-based phasing.
when_to_use_negative:
- Sequencing data is PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves
  mediocre performance on these, so a HiFi-specific assembler is preferable
- Reads are short Illumina paired-end reads — the overlap presets are tuned for long
  noisy reads
- Only approximate coordinates are needed but downstream code requires exact base-level
  CIGAR without adding the -c flag
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Detects candidate read-to-read overlaps ('-x ava-pb'/'ava-ont') and maps corrected
    reads to the first-round assembly ('-x map-pb -c -p 0.5 -r 1000'); alignments
    are consumed by PECAT's string-graph builder
  repo: https://github.com/lh3/minimap2
- name: PECAT
  role: Pipeline that extends minimap2 alignments to read ends with the diff algorithm,
    filters long overhangs, and builds the string graph from the detected overlaps
  repo: https://github.com/lemene/PECAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:92aeb9535484d6c0e10d9f8abf1716b6e34db430cf18a9c559b9a20de40387d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# minimap2-overlap-detection

## Summary

Use minimap2 to detect candidate read-to-read overlaps among long noisy reads (PacBio CLR, Nanopore R9/R10) as the first step of PECAT's string-graph assembly, and to map corrected reads onto first-round contigs for downstream heterozygous SNP calling.

## When to use

Use when long noisy reads (PacBio CLR or Nanopore R9/R10) need candidate overlaps for string-graph construction, or when corrected reads must be mapped to contigs with base-level alignment for SNP-based phasing.

## When NOT to use

- Sequencing data is PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves mediocre performance on these, so a HiFi-specific assembler is preferable
- Reads are short Illumina paired-end reads — the overlap presets are tuned for long noisy reads
- Only approximate coordinates are needed but downstream code requires exact base-level CIGAR without adding the -c flag

## Inputs

- Corrected long reads in FASTA/FASTQ (PacBio CLR, Nanopore R9 or R10; gzip'd files accepted directly)
- Raw long reads for raw-read overlap consistency checks
- First-round haplotype-collapsed assembly contigs (FASTA) as reference for read-to-contig mapping

## Outputs

- Candidate read-to-read overlaps in PAF format (overlaps.paf), optionally with cg CIGAR tags
- Read-to-contig alignments in SAM/PAF with base-level alignment for SNP calling

## How to apply

Run minimap2 with the read-overlap presets ('-x ava-pb' for PacBio CLR reads) to detect candidate read-to-read overlaps among corrected reads, supplying the align_rd2rd_options parameters from the PECAT config. In PECAT, these minimap2 alignments are then extended to the ends of the reads with the diff algorithm and overlaps still having long overhangs are filtered out (align_filter_options) before string-graph construction, where transitive edges are marked inactive so base-level identity computation is only needed on active edges. For the second assembly round, map corrected reads back to the first-round haplotype-collapsed assembly using minimap2 with '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR reads; the -c flag produces base-level CIGAR needed for SNP/allele consistency checks (strict threshold of 8 different SNP alleles on corrected reads; loose threshold of 6 on raw reads). Note that once a minimap2 index is built with -d, indexing parameters (-k, -w, -H, -I) cannot be changed during mapping, so regenerate indexes per data type.

## Related tools

- **minimap2** (Detects candidate read-to-read overlaps ('-x ava-pb'/'ava-ont') and maps corrected reads to the first-round assembly ('-x map-pb -c -p 0.5 -r 1000'); alignments are consumed by PECAT's string-graph builder) — https://github.com/lh3/minimap2
- **PECAT** (Pipeline that extends minimap2 alignments to read ends with the diff algorithm, filters long overhangs, and builds the string graph from the detected overlaps) — https://github.com/lemene/PECAT

## Examples

```
minimap2 -x ava-pb corrected_reads.fa corrected_reads.fa > overlaps.paf
```

## Evaluation signals

- Output is valid PAF/SAM from minimap2 and downstream PECAT stages proceed without missing-overlap errors
- After the diff-algorithm extension step, no retained overlap has long overhangs (align_filter_options applied)
- String graph is built with transitive edges (Myers' algorithm) marked inactive, so local alignment/identity computation runs only on the small set of active edges — wall time should stay far below full pairwise alignment
- For round two, read-to-contig mappings cover SNP sites with coverage in the range [10, 1000] to support heterozygous SNP calls
- Demo run produces $PROJECT/3-assemble/primary.fasta and round-two contig files under $PROJECT/5-assemble as valid FASTA

## Limitations

- minimap2 index parameters (-k, -w, -H, -I) are fixed once an index is built with -d; different data types require separate indexes
- Overlaps are only approximate mappings unless base-level alignment is requested (-c), which is slower
- For diploid phasing, corrected-read SNP calls alone are insufficient; raw-read SNP calling (e.g., Clair3 for Nanopore) must supplement them

## Evidence

- [discussion] we find candidate overlaps between raw reads using minimap2 with parameters: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [discussion] We map corrected reads to the first round of assembly using minimap2: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] overlaps extended to read ends and long-overhang filtering: "we extend the alignment to the ends of the reads with the diff36 algorithm and filter out overlaps still with long overhangs."
- [readme] minimap2 overlap preset usage example: [evidence span withheld — anchor status: external_doc]
