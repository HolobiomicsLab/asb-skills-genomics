---
name: polyploid-long-read-assembly
description: Use when assembling long TGS reads (PacBio or ONT) from diploid/polyploid
  or mixed samples where standard assemblers mask true low-frequency haplotype variants,
  and haplotype coverage must be quantified.
when_to_use_negative:
- The input is a haploid/clonal sample with no haplotype structure — consensus-based
  correction is sufficient and VeChat adds runtime/memory overhead
- The analysis goal is exploring optimal preprocessing/error-correction/assembly pipeline
  combinations for polyploid, cancer or metagenomic settings, which the authors state
  is beyond the scope of this study and requires further efforts
- Only short Illumina reads are available — VeChat is designed for long TGS reads,
  and existing short-read error correction tools are the appropriate class
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Haplotype-aware variation graph-based error correction of raw long reads prior
    to assembly
  repo: https://github.com/HaploKit/vechat
- name: HiCanu
  role: Haplotype-aware assembler run on corrected and raw read sets with platform-appropriate
    pb/ont settings
- name: Canu
  role: Baseline assembler (whose error-correction module uses Racon) for comparison
    against VeChat+HiCanu
- name: Flye
  role: Second, non-haplotype-aware assembler run under equivalent settings as a comparison
- name: QUAST
  role: Computes haplotype coverage, misassembly counts and error rates against the
    reference genome
- name: Minimap2
  role: Computes minimizer-based all-vs-all overlaps and base-level alignments inside
    VeChat's two correction cycles
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:9e859f0df383c7beccf449eb0c7689d237802a16a085825fe6bb98b9a82e4423
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Canu
  - Flye
  - HiCanu
  - Minimap2
  - QUAST
  - VeChat
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# polyploid-long-read-assembly

## Summary

Assemble haplotype-aware genomes from long reads by first correcting reads with VeChat (variation graph-based error correction) and then running haplotype-aware and standard assemblers (HiCanu/Canu, Flye), followed by QUAST evaluation of haplotype coverage and error rates.

## When to use

Use when assembling long TGS reads (PacBio or ONT) from diploid/polyploid or mixed samples where standard assemblers mask true low-frequency haplotype variants, and haplotype coverage must be quantified.

## When NOT to use

- The input is a haploid/clonal sample with no haplotype structure — consensus-based correction is sufficient and VeChat adds runtime/memory overhead
- The analysis goal is exploring optimal preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer or metagenomic settings, which the authors state is beyond the scope of this study and requires further efforts
- Only short Illumina reads are available — VeChat is designed for long TGS reads, and existing short-read error correction tools are the appropriate class

## Inputs

- Raw long-read FASTA/FASTQ read sets (PacBio or ONT) for diploid or polyploid datasets
- VeChat-corrected reads (reads.corrected.fa, FASTA) from the upstream correction step
- Reference genome FASTA for QUAST evaluation

## Outputs

- Haplotype-aware genome assemblies from HiCanu/Canu and Flye on corrected and raw reads
- QUAST metrics table (quast_report.tsv) with haplotype coverage, misassembly counts and assembly error rates per assembler and read type

## How to apply

Correct raw long reads with VeChat (clone github.com/HaploKit/vechat), optionally tuning edge retention via --min-confidence/-d (default 0.2), --min-support/-s (default 0.2) and --scrub for chimeric reads; VeChat runs two cycles, first computing Minimap2 minimizer all-vs-all overlaps and building/pruning ad-hoc variation graphs with POA, then computing base-level alignments enabling haplotype-aware read overlap filtration. Assemble both the VeChat-corrected reads (e.g. reads.corrected.fa) and the matching raw FASTA/FASTQ sets with HiCanu/Canu using the platform-appropriate setting (pb or ont), and with Flye as a non-haplotype-aware comparator. Run QUAST on every assembly against the corresponding reference genome to compute haplotype coverage (HC), misassembly counts and error rates. The rationale is that graph-based templates capture variants of lower frequency that consensus-based self-correction mistakenly masks as errors; success is judged by HC gains (e.g. +7.6% diploid, +14% tetraploid) and >90x lower error rate for VeChat+HiCanu over Canu.

## Related tools

- **VeChat** (Haplotype-aware variation graph-based error correction of raw long reads prior to assembly) — https://github.com/HaploKit/vechat
- **HiCanu** (Haplotype-aware assembler run on corrected and raw read sets with platform-appropriate pb/ont settings)
- **Canu** (Baseline assembler (whose error-correction module uses Racon) for comparison against VeChat+HiCanu)
- **Flye** (Second, non-haplotype-aware assembler run under equivalent settings as a comparison)
- **QUAST** (Computes haplotype coverage, misassembly counts and error rates against the reference genome)
- **Minimap2** (Computes minimizer-based all-vs-all overlaps and base-level alignments inside VeChat's two correction cycles) — https://github.com/lh3/minimap2

## Examples

```
vechat --scrub -d 0.2 -s 0.2 example/reads.fq.gz
```

## Evaluation signals

- Haplotype coverage (HC) from QUAST should increase with VeChat correction: reported +7.6% on diploid (92% → 99.6% with HiCanu) and +14% on tetraploid
- VeChat+HiCanu assembly error rate should be >90x lower than Canu, and misassembly counts should be fewer than for raw-read assemblies
- Corrected reads should show reduced error rates relative to other correctors (VeChat reports 4–15x lower error rates for PacBio and 1–10x for ONT vs state of the art)
- Variation-graph pruning converges: the spurious node/edge removal followed by re-alignment loop must be repeated until convergence before deriving consensus sequences
- Edge filtering parameters remain at documented defaults (-d 0.2, -s 0.2, consensus-round overlap identity 0.99) unless explicitly justified

## Limitations

- VeChat does not win in computational resources: on large datasets it has longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads; edit-distance and sequence-to-graph alignments could be replaced by more efficient routines
- Exploration of optimal pipeline combinations (preprocessing + error correction + assembly) for polyploid, cancer or metagenomic settings is beyond the scope of the study and requires further efforts

## Evidence

- [abstract] Using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies.: "using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies"
- [abstract] Variation graph template rationale.: "Unlike consensus sequence templates, graph-based reference systems are not affected by such biases, so do not mistakenly mask true variants as errors"
- [methods] Minimum confidence parameter for graph edges.: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [methods] Chimeric read scrubbing option.: "--scrub               scrub chimeric reads (default: False)"
- [results] Read alignment pile windowing in cycle 1.: "the read alignment pile is divided into small segments, where each of the segments gives rise to a window like part of the pile"
- [results] Haplotype-aware filtration in cycle 2.: "In step 1, however, beyond computing all-vs-all overlaps, base-level alignments are computed, which enables haplotype aware read overlap filtration."
- [discussion] Runtime and memory limitation.: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [readme] README description of graph-based error vs variant discrimination.: "VeChat distinguishes errors from haplotype-specific true variants based on variation graphs, which reflect a popular type of data structure for pangenome reference systems."
