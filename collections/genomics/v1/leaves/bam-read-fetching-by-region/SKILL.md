---
name: bam-read-fetching-by-region
description: Use when haplotype-resolved long-read BAM files (e.g., minimap2-aligned
  ONT or PacBio alignments) and a BED file of candidate repeat regions are available
  and reads fully spanning each region must be extracted for POA consensus building.
when_to_use_negative:
- The candidate region exceeds the long-read length such that no sequencing read entirely
  spans R = [S, E]; the fetch-and-trim step requires full-spanning reads
- Alignments are not haplotype-resolved; TRiCoLOR requires haplotype-resolved long-read
  alignments as input
- The target is an unassembled genomic region (e.g., human centromeres and telomeres),
  where TRiCoLOR cannot identify repeats
tools:
- name: TRiCoLOR REFER
  role: Module that fetches fully spanning reads from haplotype-specific BAM files
    and trims them to the region coordinates S-E before POA
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: SIMD-accelerated partial order alignment framework run on the trimmed haplotype-specific
    read sets to compute the consensus per region and haplotype
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:6dab6967b9dc92a0b270aed2d08861926dc75cc031e3b3c717387d3ad13744b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# BAM read fetching by region

## Summary

Fetch, from haplotype-specific long-read BAM alignments, all sequencing reads that entirely span a candidate repetitive region R = [S, E] listed in a BED file, and trim each read so only the sequence between S and E is retained, preparing haplotype-specific read sets for partial order alignment (POA) consensus generation.

## When to use

Use when haplotype-resolved long-read BAM files (e.g., minimap2-aligned ONT or PacBio alignments) and a BED file of candidate repeat regions are available and reads fully spanning each region must be extracted for POA consensus building.

## When NOT to use

- The candidate region exceeds the long-read length such that no sequencing read entirely spans R = [S, E]; the fetch-and-trim step requires full-spanning reads
- Alignments are not haplotype-resolved; TRiCoLOR requires haplotype-resolved long-read alignments as input
- The target is an unassembled genomic region (e.g., human centromeres and telomeres), where TRiCoLOR cannot identify repeats

## Inputs

- Haplotype-resolved long-read alignment files (BAM) for each sample/haplotype
- BED file listing candidate repetitive regions as R = [S, E] (e.g., SENSoR-detected low-entropy regions)
- Optional: ONT FASTQ or PacBio alignments (HGSVC data were minimap2-aligned to GRCh38)

## Outputs

- Per-region, per-haplotype sets of reads trimmed to coordinates S-E
- consensus_sequences.fasta containing one low-error consensus sequence per candidate region per haplotype

## How to apply

For each region R = [S, E] in the input BED file, query the haplotype-specific BAM files for sequencing reads that entirely span the region, since TRiCoLOR's REFER module profiles TRs from haplotype-resolved long-read alignments. Trim each fetched read so that only the subsequence between coordinates S and E remains; this trimming directly reduces the runtime of the downstream POA step. Feed the trimmed haplotype-specific read sets to SPOA, the SIMD-accelerated partial order alignment framework, to compute a highly accurate consensus per region and haplotype. Write one consensus per candidate region per haplotype to consensus_sequences.fasta. Expect SPOA to yield an approximate error reduction of ~77% for ONT and ~88% for PB reads relative to raw error-prone long reads.

## Related tools

- **TRiCoLOR REFER** (Module that fetches fully spanning reads from haplotype-specific BAM files and trims them to the region coordinates S-E before POA) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (SIMD-accelerated partial order alignment framework run on the trimmed haplotype-specific read sets to compute the consensus per region and haplotype) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Every fetched read entirely spans the region, i.e., covers both S and E of the BED interval
- Trimmed reads have length E - S + 1 bp, containing only the sequence between coordinates S and E
- consensus_sequences.fasta contains exactly one consensus entry per candidate region per haplotype
- Consensus error rate is reduced by approximately 77% for ONT and 88% for PB reads relative to the raw long reads
- Consensus computation succeeds on regions with mean coverage passing the article's filters (coverage >8 for ONT, >10x for PB)

## Limitations

- No consensus can be generated for a region unless at least one read entirely spans it; insufficient read lengths failing to encompass entire expanded repeats are a known challenge
- TRiCoLOR does not support polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) cannot be detected

## Evidence

- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [readme] TRiCoLOR: tandem repeats profiler for long-read sequencing data: "TRiCoLOR: tandem repeats profiler for long-read sequencing data"
