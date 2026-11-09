---
name: haplotype-aware-consensus-generation
description: Use when correcting long noisy TGS reads (PacBio/ONT FASTA/FASTQ) from
  mixed samples—metagenomes or polyploid genomes—where a plain consensus template
  would mask low-frequency haplotype variants as errors.
when_to_use_negative:
- Input reads are diploid-only and already corrected — earlier approaches like Racon,
  Canu or FLAS consensus-template correction suffice
- The task is optimizing runtime/peak memory on very large datasets — VeChat has longer
  runtimes and higher peak memory usage than competitors (see --split thresholds for
  FASTQ > 20G or FASTA > 10G)
- Choosing preprocessing/error-correction/assembly pipeline combinations for polyploid,
  cancer or metagenomic settings beyond running correction itself — the authors state
  this requires further efforts beyond the study
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: VeChat
  role: Runs the two-cycle variation-graph correction pipeline; cycle 2 derives haplotype-aware
    consensus sequences from unpruned graphs using dynamic programming
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps; in cycle 2 with base-level
    alignments enabling haplotype-aware read overlap filtration
  repo: https://github.com/lh3/minimap2
- name: yacrd
  role: Supports chimeric read scrubbing (--scrub) by detecting chimeras from all-against-all
    read mapping pile-up coverage
  repo: https://github.com/natir/yacrd
- name: fpa
  role: Filters all-against-all pairwise alignments (internal match, containment,
    dovetail, overlap length) used in the overlap-processing steps
  repo: https://github.com/natir/fpa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:27f7acadafeaba8973df66503f0f468a7ad1cb100841509c0dc76d96f6fba0a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Racon
  - VeChat
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# haplotype-aware-consensus-generation

## Summary

Generates haplotype-aware error-corrected long reads by deriving consensus sequences from variation graphs (rather than a single consensus template) with a dynamic programming algorithm, preserving true low-frequency variants that consensus-sequence methods would mask as errors.

## When to use

Use when correcting long noisy TGS reads (PacBio/ONT FASTA/FASTQ) from mixed samples—metagenomes or polyploid genomes—where a plain consensus template would mask low-frequency haplotype variants as errors.

## When NOT to use

- Input reads are diploid-only and already corrected — earlier approaches like Racon, Canu or FLAS consensus-template correction suffice
- The task is optimizing runtime/peak memory on very large datasets — VeChat has longer runtimes and higher peak memory usage than competitors (see --split thresholds for FASTQ > 20G or FASTA > 10G)
- Choosing preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer or metagenomic settings beyond running correction itself — the authors state this requires further efforts beyond the study

## Inputs

- Pre-corrected reads from cycle 1 (concatenated target subreads)
- Variation graphs from cycle 1 (unpruned, built via POA from read alignment pile windows)
- Raw read file (FASTA/FASTQ, gzip-compressed allowed, e.g. example/reads.fq.gz)
- Sequencing platform flag (--platform pb or ont)

## Outputs

- Final corrected reads in reads.corrected.fa (FASTA, one corrected sequence per input read)
- Haplotype-aware consensus sequences derived per target read via dynamic programming

## How to apply

In cycle 2 of VeChat, first recompute all-vs-all overlaps with base-level alignments, applying haplotype-aware read overlap filtration using a minimum sequence identity of 0.99 in the consensus round (--min-identity-cns). Then derive haplotype-aware consensus sequences from the variation graphs constructed in cycle 1 — crucially, the graphs are NOT re-pruned in this cycle — using a dynamic programming algorithm. Concatenate the per-window consensus sequences for each target read and join all target reads to produce the final corrected set (reads.corrected.fa). The graph-based template is what prevents true low-frequency variants from being mistaken for sequencing errors, unlike Racon/Canu/FLAS-style consensus templates.

## Related tools

- **VeChat** (Runs the two-cycle variation-graph correction pipeline; cycle 2 derives haplotype-aware consensus sequences from unpruned graphs using dynamic programming) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps; in cycle 2 with base-level alignments enabling haplotype-aware read overlap filtration) — https://github.com/lh3/minimap2
- **yacrd** (Supports chimeric read scrubbing (--scrub) by detecting chimeras from all-against-all read mapping pile-up coverage) — https://github.com/natir/yacrd
- **fpa** (Filters all-against-all pairwise alignments (internal match, containment, dovetail, overlap length) used in the overlap-processing steps) — https://github.com/natir/fpa

## Examples

```
vechat --platform pb --min-identity-cns 0.99 -d 0.2 -s 0.2 example/reads.fq.gz  # output: reads.corrected.fa
```

## Evaluation signals

- Output file reads.corrected.fa exists and is a valid FASTA containing exactly one corrected sequence per input read
- Corrected reads contain 4 to 15 times fewer errors for PacBio and 1 to 10 times fewer for ONT compared to state-of-the-art tools
- Error rates on simulated diploid/triploid/tetraploid PacBio CLR reads are approximately 14–30x, 9–26x and 4–11x lower than competing methods
- Switch error and QV improve on benchmark genomes such as HG002 (~1.5x lower error rate and ~1.2x lower switch error vs. CONSENT)
- Low-frequency haplotype variants in mixed samples (e.g. 5-strain HIV mixture, yeast pseudo-diploid mock community) are retained rather than masked as errors

## Limitations

- VeChat does not win on computational resources; on large datasets it experiences longer runtimes and higher peak memory usage
- Use --split for large inputs (FASTQ > 20G or FASTA > 10G) and --scrub to remove chimeric reads
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads; edit-distance and sequence-to-graph alignments could be replaced by more efficient routines
- Optimal combinations of preprocessing, error correction and assembly for polyploid, cancer or metagenomic settings are beyond the study's scope

## Evidence

- [results] haplotype-aware consensus sequences derived from unpruned variation graphs by dynamic programming: "(haplotype-aware!) consensus sequences (displayed as thick yellow arrows in Fig. 3) are derived from the constructed variation graphs by using a dynamic programming algorithm"
- [results] cycle 2 base-level alignments enable haplotype-aware read overlap filtration: "base-level alignments are computed, which enables haplotype aware read overlap filtration"
- [methods] minimum sequence identity threshold in the consensus round: "minimum sequence identity between read overlaps in the consensus round (default: 0.99)"
- [results] joining target reads produces final output: "After concatenating these consensus sequences in step 5, joining all target reads in step 6 generates the final output of Vechat."
- [readme] graph-based template distinguishes errors from true haplotype variants: "VeChat distinguishes errors from haplotype-specific true variants based on variation graphs, which reflect a popular type of data structure for pangenome reference systems."
